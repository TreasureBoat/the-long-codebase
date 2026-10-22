---
layout: post
title: "We keep the old deploy path as bait"
subtitle: "Our framework still builds the WebObjects-shaped deployment. Nothing needs it any more — we keep it because WebObjects developers recognize it, and I am not at all sure that works"
date: 2026-09-05 17:00:00 +0900
author: ken
description: "Why a framework keeps a deployment model it has already superseded, what that actually costs, and the failure mode in it that takes an afternoon to find."
excerpt: >-
  Our framework packages an application three ways. The oldest is the WebObjects shape —
  a bundle on one server, its static files on another, Apache joining them. The newer two
  are better by every measure I care about. We still build the old one, and the reason is
  not technical.
cta_hook: "If you have kept an old path alive to make newcomers comfortable, I would like to know whether anyone actually used it."
---

Our framework can package an application three ways. The newest two are better than the
oldest by every measure I care about: fewer moving parts, less required of the server,
no second machine to keep in step.

We still build the oldest one. It will be removed some day. It is not being removed yet,
and the reason has nothing to do with the code.

The reason is that it is the deployment model WebObjects developers already know, and I
would like some of them to come over.

## What the old model is

If you deployed WebObjects, this needs no explanation. A `.woa` bundle goes on the
application server: JARs, resources, components, run scripts. A second bundle — same
name, same timestamp — goes on the web server, holding nothing but static files. Apache
serves the second straight from disk and reverse-proxies everything else to the running
application.

It was a good design and parts of it still are. Static files never touch the Java
process. The version and build timestamp sit in the URL path, so every release is
cache-busted by construction — cache-busting before the term existed. Rolling back means
pointing Apache at yesterday's folder.

It also has a discipline you cannot opt out of: **both halves must carry the same
timestamp.** Rebuild one without the other and the application emits URLs naming one
folder while Apache serves another. The application works perfectly. Every stylesheet
404s. No log mentions a timestamp.

## The failure that costs an afternoon

One property decides whether that setup works, and it is not in Apache.

The framework chooses at runtime what kind of URL to emit for a stylesheet. If the
request looks like it came through a web server, it emits a path for the web server to
serve. Otherwise it emits a URL that makes the application serve the file itself, out of
the JAR.

Deploy behind a plain reverse proxy — no static bundle copied anywhere — and the request
still *looks* like it arrived through a web server, because that check is a header
check. So the application confidently emits web-server URLs for files nobody copied to
the web server.

You get a working application with no styling at all. Apache is right. The application
is right. The logs are clean. The fix is one property; finding out which property is the
afternoon.

Since the check is header-based, there is a second case: a cloud load balancer that adds
its own tracing header makes a direct request look proxied too.

## What keeping it costs

Less than I expected, which is the only reason it is still here.

It is not a special case threaded through the framework. It is a build type and a
property, and the code choosing between them is one method. Old deployment paths usually
cost you by leaking into everything; this one stayed in one place.

What it does cost is documentation. Three models means three sets of instructions, and
the failure above is only findable if somebody wrote down that the property exists. That
is most of why the long version got written.

## The part I am unsure about

The case for keeping it is that a WebObjects developer looking at TreasureBoat sees a
deployment model they have run for fifteen years, and one fewer unfamiliar thing between
them and trying it.

The case against is that I might have the psychology exactly backwards. People who are
still running WebObjects in 2026 are, by revealed preference, people who are content
with what they have. A familiar deployment story may not be what moves someone like
that, and if it is not, then this is a path kept alive for an audience that was never
going to walk down it.

I do not know which of those is true. It costs a build type and a property, so I am
willing to be wrong for a while longer. But it is worth being clear with myself that
this is a marketing decision wearing engineering clothes, and that the newer models are
what I would tell anyone starting today to use.

## The long version

The full walkthrough — both builds, the timestamp rule, the Apache configuration, the
property and its two settings — is on our documentation blog:

[Legacy Deploy — NLB + WSR behind Apache](https://blog.treasureboat.org/deploy/2026/09/05/deploy-nlb-wsr.html)

The two newer models get the same treatment over the coming days and I will link them
here as they go up. In short: one carries every dependency with it, for locked-down or
air-gapped servers; the other carries almost nothing and lets Maven fetch the rest.
