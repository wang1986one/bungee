# Bungee: Audio Time-Stretching & Pitch-Shifting Library

Bungee is a modern, open-source C++ library for high-quality audio time-stretching and pitch-shifting in real-time or offline. Easily integrate advanced audio timescale processing into your application.

Bungee can adjust the speed of audio without affecting pitch; transpose audio pitch without affecting speed; or any combination of playhead position and pitch manipulation.
* Simple, fast phase-vocoder-based algorithm with good quality audio output ([🎧  hear some comparisons](https://bungee.parabolaresearch.com/compare-audio-stretch-tempo-pitch-change.html) with other algorithms)
* Modern C++ for clean and resilient code

Bungee is unique in its controllability, allowing continually changing audio tempo and pitch manipulation with seamless support of zero and negative playback speeds. So it can be used for a "smooth scrub" or for rendering lifelike audio for slow-motion videos.

 ⭐️ _To support Bungee, please consider [giving this repo a star](https://github.com/bungee-audio-stretch/bungee/stargazers)_ .

![GitHub Release](https://img.shields.io/github/v/release/bungee-audio-stretch/bungee)
![GitHub License](https://img.shields.io/github/license/bungee-audio-stretch/bungee)
![GitHub Downloads](https://img.shields.io/github/downloads/bungee-audio-stretch/bungee/total)
![GitHub Repo stars](https://img.shields.io/github/stars/bungee-audio-stretch/bungee)

## Getting started with Bungee Audio Time-Stretching

Bungee's dependencies are managed as git submodules; so clone like this:
```
git clone --recurse-submodules https://github.com/bungee-audio-stretch/bungee
```

Use CMake to configure and build the bungee library and command-line executable:
```
cd bungee
mkdir build && cd build
cmake ..
cmake --build .
```

After a successful build, run the bungee executable
```
./bungee --help
```

## Integrating Bungee C++ Audio Library

Every commit pushed to this repo's main branch is automatically tagged and built into a release. Each release contains Bungee built as a shared library together with headers, sample code and a sample command-line executable that uses the shared library. Releases support common platforms including Linux, Windows, MacOS, Android and iOS.

## Example: Real-Time Audio Pitch-Shifting in C++

Bungee operates on discrete, overlapping "grains" of audio, typically processing around 100 grains per second. Parameters such as position and pitch are provided on a per-grain basis so that they can be changed continuously as audio rendering progresses. This means that only minimal parameters are required for  instantiation.

For a working example of this API, see  [cmd/main.cpp](./cmd/main.cpp).

### Instantiation

To instantiate, include the [bungee/Bungee.h](./bungee/Bungee.h) header file, create a `Stretcher<Basic>` object and initialise a `Request` object:

``` C++
#include "Bungee.h"
#include <cmath>

const int sampleRate = 44100;

Bungee::Stretcher<Bungee::Basic> stretcher({sampleRate, sampleRate}, 2);

Bungee::Request request{};

// Set pitch, this example shows an upward transposition of one semitone.
request.pitch = std::pow(2., 1. / 12.);

// Set initial speed, this example shows how to achieve constant 75% output speed.
request.speed = 0.75;

// Set initial starting position at 0.5 seconds offset from the start of the input buffer.
request.position = 0.5;

// This call adjusts request.position so that stretcher's pipeline will be fully initialised by the
// time it reaches the starting position of 0.5 seconds offset.
stretcher.preroll(request);
```

### Granular loop

`Stretcher`'s processing functions are typically called from within a loop, each iteration of which corresponds to a grain of audio. For each grain, the functions `Stretcher<Basic>::specifyGrain`, `Stretcher<Basic>::analyseGain` and `Stretcher<Basic>::synthesiseGrain` should be called in sequence.
```C++
while (true)
{
    // ...
    // Change request's members, for example, position, speed or pitch, as required here.
    // ...
 
    auto inputChunk = stretcher.specifyGrain(request);

    // ...
    // Examine inputChunk and retrieve the segment of input audio that the stretcher requires here.
    // Set data and channelStride to point to the required segment of input data.
    // ...

    stretcher.analyseGrain(data, channelStride);

    Bungee::OutputChunk outputChunk;
    stretcher.synthesiseGrain(outputChunk);

    // ...
    // Output the audio buffer indicated by outputChunk here.
    // ...

    // Prepare request for the next grain (modifies request.position according to request.speed)
    stretcher.next(request);
}
```

### Things to note

* Bungee is most commonly used for stereo and mono operation at sample rates of 44.1kHz and 48kHz. In principle, though, any practical sample rate and number of audio channels are supported.

* `Request::position` is a timestamp, it defines the grain centre point in terms of an input audio frame offset. It is the primary control for speed adjustments and is also the driver for seek and scrub operations. The caller is responsible for deciding  `Request::position` for each grain. 

* The caller owns the input audio buffer and must provide the audio segment indicated by `InputChunk`. Successive grains' input audio chunks may overlap. The `Stretcher<Basic>` instance reads in the input chunk data when `Stretcher<Basic>::analyseGrain` is called.

* The `Stretcher<Basic>` instance owns the output audio buffer. It is valid from when `Stretcher<Basic>::synthesiseGrain` returns up until `Stretcher<Basic>::synthesiseGrain` is called for the subsequent grain. Output audio chunks do not overlap: they should be concatenated to produce an output audio stream.

* Output audio is timestamped. The original `Request` objects corresponding to the start and end of the chunk are provided by `OutputChunk`.

* Bungee works with 32-bit floating point audio samples and expects sample values in the range -1 to +1 on both input and output. The algorithm performs no clipping.

* When configured for 1x speed and no pitch adjustment, the difference between input and output signals should be small: minor windowing discrepencies only.

* Any special or non-numeric float values such as NaN and inf within the input audio may disrupt or cause loss of output audio.

## Dependencies

The Bungee library gratefully depends on:
* The Eigen C++ library for buffer management and mathematical operations on vectors and arrays
* The PFFFT library for Fast Fourier Transforms

The sample `bungee` command-line utility also uses:
* cxxopts library for parsing command-line options

See this repo's [.gitmodules](.gitmodules) for versioned links to these projects.

## License

Bungee is permissively licensed under the Mozilla Public License Version 2.0.

## Support

Bungee's goal is to be the _best open source audio timescale manipulation library_ available. User feedback is invaluable: please use Github issues to report anything that could be improved.

> ## Bungee Pro
> 
> Bungee Pro is a commercial product providing an upgrade path from this open-source project. It uses novel algorithms for sharp and clear professional-grade audio and runs at least as fast as Bungee, thanks to platform-specific performance optimisations.
> 
> Whilst open-source Bungee aims to be the best open-source audio time stretch algorithm, the goal of Bungee Pro is to be the _best algorithm available commercially_.
> 
> * Novel processing techniques that deliver crisp transients and preserve vocal and instrumental timbre
> * Adaptive to all genres of speech, music and sound with subjective transparency up to infinite time stretch
> * Performance optimisations for:
>    * Web AudioWorklet with SIMD128 WebAssembly
>    * Arm NEON for Android, iOS and MacOS
>    * x86-64 SIMD for Linux, Windows and MacOS
> * A ready-to-use Web Audio implementation 
> * Professional support
>
> Bungee Pro is used in a wide variety of applications including movie post production software, educational apps and musicians' tools. 
>
> [Listen to an extensive evaluation](https://bungee.parabolaresearch.com/compare-audio-stretch-tempo-pitch-change.html) of Bungee and Bungee Pro against state-of-the-art techniques.
>
> | ![waveform animation](./README.md-waveform.png) | Try the [WebAssembly demo](https://bungee.parabolaresearch.com/change-audio-speed-pitch.html)  of Bungee Pro in your browser.|
> |--|--|
>
