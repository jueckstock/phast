# phast: Play-Head Acceleration and Sandboxing Technology (Concept)

> Memo-to-self RE: a conceptual software framework for accelerating
> and stabilizing frame-by-frame seeking across video files (such
> as during non-linear video editing).

## Abstract

Proposing `phast` as a library/framework for *sandboxing* third-party video codecs
inside isolated worker processes and *accelerating* access to video content through
a pipelined, shared-memory random-frame-access API with soft-realtime latency
guarantees (i.e., it will _always_ show you a video frame at the appointed time;
it may be a _blank_ frame [the fallback placeholder], or the nearest I-frame [compromise placeholder], or the actual frame [best-effort deliverable]).

A secondary goal of `phast` would be to provide for *transparent proxification* of video,
downsampling frames and keeping them around in a large (on disk) LRU cache (which could,
if storage is available, be a full proxy copy of the original video).

The above features must be delivered at relatively low cost, as they would be
the *foundation* on which a high-overhead video editor/compositor would be built.
The nutshell performance goal:
**an endless-random-frame-seek "torture test" should consume no more than 10% of the sustainable TDP "budget" of the host system.**

## Motives

As a category, FOSS video editors do not have a reputation for stability or performance (or quality in general, but that is another topic).

### Stability

A frequent complaint is instability: many FOSS video editors are known to crash frequently.
(See basically any discussion of FOSS NLEs on Hacker News.)
Crashes lose precious work, or at the very least massively disrupt precious workflow.
So crashes are bad.

**Hypothesis:** many (most?) crashes in FOSS NLEs are caused by bugs in the underlying video
streaming frameworks/layers (e.g., GStreamer, MLT) or even individual video codec
implementations.  "Bullet-proof" video codecs and frameworks (perhaps made so by using modern
low-overhead memory-safe languages) are a great idea, but we need a better solution *now*.

**Proposal:** follow the render-process sandboxing model used by modern web browsers
and isolate all video decoding/sampling/caching logic in separate processes controlled via
low-latency (probably shared-memory) APIs.  If this "play-head" process crashes, the
main editor process is uneffected and may not even know, if the framework automatically
restarts another play-head process and resumes at the same location.

### Performance & Consumer-Grade Content

Performance is another issue with FOSS editors, especially when they attempt to support
direct editing of lossy, non-constant-framerate video data (e.g. h.264).
In one sense, this is not a problem, just a fact of life: there's a reason the professionals
use non-lossy formats and/or edit using lower-res "proxy" files.

However, consumer cameras generally produce lossy video.
Transcoding (including proxy-generation) is a valid option but an extra step that
raises the barrier to entry for casual users and keeps tools in the nerd-ghetto,
where they will never have a large enough community to get really good.

It's fine to imagine your FOSS NLE doesn't need to handle consumer-friendly inputs
because you dream of it being a real professional tool (and hey, pro tools don't
handle consumer-friendly inputs well, either!)

But the history of tech is one of *upward market segment creep*: it's the cheap
hobbiest tools that build a big consumer following and then get good enough to go pro.
It's almost never the other way around, with big pro tools "dumbing down" to
reach the consumer/hobby mass market.

**Hypothesis**: the path to success and stability for FOSS NLEs is to provide simple but solid
(i.e., fast, stable, adaptive to typical consumer inputs) features to a broad, entry-level
audience, then leverage that audience to "creep up" into higher-level market segments.

**Proposal**: extend the `phast` play-head process model to support transparent
proxy rendering/caching (on a frame-by-frame basis) so that "proxies" can be
created on the fly and can even use LRU cache semantics to keep only "hot"
frames around on disk.