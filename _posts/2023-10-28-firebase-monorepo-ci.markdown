---
layout: post
title:  "Move fast! Break things! Get stuck!"
date:   2023-10-28 23:00:00 -0700
categories: monorepo firebase continuous-integration github
mermaid: true
---

When I started on my journey of building [Datayaki][datayaki], I chose to heavily bias towards fast and scrappy feature development over engineering rigor. This was a conscious decision to counteract what I encountered while building Remix, aka Mix.byCollab.com half a decade ago.

When I started working on Remix, I had just left Google, and took a lot of the great learnings from Google and applied it towards the project. I built a bazel monorepo. I looked into CI/CD and Code Review solutions like Gerrit, since none existed on Github at the time. I also went down the rabbit hole of setting up a Kubernetes cluster, and designed a service proxy framework so that my backend services, and my javascript frontend can communicate using protobufs, grpc, http2, and envoy sidecar proxies. I frigging' got bazel to build and test my polyglot project all at once with a simple `bazel build ...` command. Do you know how friggin' hard that is?

I did all that and more, except for iterating on the product! I spent 6 months down the drain back then chasing those engineering rigor problems. Of course, I did build a basic collaborative video editor on the web, but then when I brought it to the first customer, it bombed. And around the same time I ran out of money due to an unexpected tax liability... and the project didn't see the light of day. I'll save that sad story for another day.

I didn't wanna end up making the same mistakes from the past this time around with Datayaki. I wanted to iterate on product features quickly and put it in the hands of potential customers to get their feedback. So, I basically went against all my 20 years of software engineering training, and went all in on the Amazon way of scrappily putting together the bare minimum effort to get things out, and only investing more time when things stick and scale and rigor graduate from being nice-to-haves, and turn into necessities.

So, I started with just one choice -- that my repository would be a monorepo. This is the one thing I wasn't willing to budge on, as I've seen how well companies large and small get things done fast with a monorepo, vs. every time/service maintaining its own repo. But of course, this too was done in a scrappy way. It was a monorepo only to the point that there was a single repo with a folder structure that separated out different areas of concern. There was no unified build system. No integration tests. Not even unit tests. No unified deployment system. It was just a file/folder structure, and code pushed to a single github repository.

Everything else was done manually, and on an as neeeded basis. There were no automated processes. For the backend, I chose Firebase Cloud Functions... and written in Typescript. No polyglot dickery this time. There's one language across the code base... Both front-end and back-end run on Typescript, so that I only have to find solutions for weird problems once, and also don't have to switch context in my brain between different parts of the code. Beyond that, everything was manual. Experimenting with different features? manual. Deploying to staging environment? manual. Deploying to prod? manual. Setting the version number? manual. Testing? Non existent!!!😱

This was great for the first 3 months, when I had almost no usage and a few signups here and there. But then, this past month I found it excrutiatingly hard to move the needle on the product... especially after a 2 month hiatus on product development in between, due to working on other people's product ideas as I was trying to find a suitable co-founder (Basically, 2 months spent dickin around from Datayaki's perspective).

I was extremely worried that this may be a case of me lacking motivation on the product and that the dream that was Datayaki was meant to disappear... But, that wasn't really the case. I knew it wasn't the case because I'm still passionately dreaming up new features to build, and reaching out to people and listening to their data analytics woes and needs. It wasn't lack of motivation. It was friction! I was stuck in a rut because of how bad of a state my uncommitted codebase was in, and how I couldn't deploy anything because there is no workable state I could baseline my code back to. Scrappiness has found its limit! I have to move the needle a bit on improving rigor so that the prime objective of being scrappy is achieve! i.e, Improve iteration speed.

As I was trying to iterate last month, I kinda hit a snag with the data types used by both the front-end and back-end being out of sync. So, I decided to refactor it out into a separate library `datayaki-types` which I then linked to locally. This kinda worked until I kept forgetting to build the shared library, and running pnpm install on the consuming packages. Added to that, firebase emulators were not starting up properly.

So, I spent the past week doing 3 things:

1. Bring the codebase to a buildable deployable state, and improve the build process.
2. I wanted to have a tag and release process, so that I can always go back to stable code when things break.
3. I wanted a CI/CD pipeline, so that this was done automagically for me, and I can constantly deploy and test out versions in my staging environment.

Item 1 just required tedious work and a lot of internet sleuthing and troubleshooting. 2 and 3 however, needed me to implement a bit of rigor. I spent the last 3 days sleepless to get a build and deploy system working across my monorepo. It is still just doing the bare minimum... i.e, it builds and it deploys. There are still no tests written yet. But now, I simply run `pnpm -r build` from the top of my src folder, and PNPM Workspace project takes care of building the entire project for me. How neat? Of course, life isn't always sweet to you, and now I couldn't deploy my cloud functions to firebase... because of course, Google doens't deem it important to support PNPM workspaces.

The company that introduced the industry to monorepos, does not think supporting monorepo build systems is a priority for firebase deployments!!! And they wonder why GCP is having such a tough time finding customers!

Firebase cloud function deployments happen in an interesting way. The CLI (firebase-tools) first builds the package locally using whatever build/package system you specify (In my case, PNPM). This step works just fine! Then once it verifies that the thing builds fine, it packages the source folder of the cloud function into a tarball and uploads it to Firebase's cloud. It then unpacks the tarball in the cloud, and re-builds the binary to start hosting the function. And at this point, PNPM or NPM would go and fetch all the dependencies from the cloud as NPM would normally do with node_modules... and this is where my monorepo build would fail because my local monorepo dependencies are not accessible on the cloud, and the build on the cloud has no way to dereference my `workspace:*` location for my `datayaki-types` package.

Anyway, we finally get to the meat of this blog post after my loooooong rant about all things non-meaty.

I got it to work! And this is how I did it.

In firebase.json, you can specify a pre-deploy step. By default it just does the build step. In my case, I make a clone of my functions source folder, along with the build library (called `lib` for firebase functions) and the package.json file. I dereference the local package dependencies in my package.json and then copy those libraries over to the clone. I then upload this cloned folder (which includes the dereferenced and copies over shared libraries as well) as the tarball instead of the original source folder. I use the npm package [`isolate-package`][isolate] for the cloning and dereferencing step.

Of course, OSS always comes with its share of bugs, and so does `isolate-package` where it only copies `lib/index.js` and does not copy over any other files from your build. God forbid you had a little bit of code organizational skill and split your cloud function into multiple files... it basically will ignore every other local file dependency. So, my `pre-deploy` step does a `copy everyghing/from/lib to/isolate` step in addition to running `pnpm build`, and `pnpx isolate-package`. Voila! It works.

With that, I added a couple of github workflows. The first one automatically builds the workspace and deploys my frontend and backend to my staging firebase project. Once it successfully deploys, it also creates a git tag for the release. The second workflow is a manually triggered one that lets my select a tag and deploy that version to prod.

I gotta say, these steps have already had a marked impact on the ease of adding new features and deploying code.

Look at my github contributions and guess where I introduced my build and deploy system. (HINT: It's around this blog post's posting date)

<script
  src="https://cdn.rawgit.com/IonicaBizau/github-calendar/gh-pages/dist/github-calendar.min.js"
>
</script>

<!-- Optionally, include the theme (if you don't want to struggle to write the CSS) -->
<link
  rel="stylesheet"
  href="https://cdn.rawgit.com/IonicaBizau/github-calendar/gh-pages/dist/github-calendar.css"
/>

<!-- Prepare a container for your calendar. -->
<div class="calendar">
    <!-- Loading stuff -->
    Loading the data just for you.
</div>

<script>
    new GitHubCalendar(".calendar", "thekumar");
</script>

[isolate]:https://www.npmjs.com/package/isolate-package
[datayaki]:https://datayaki.com