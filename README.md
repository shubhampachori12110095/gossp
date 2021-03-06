# GOSSP

[![Build Status](https://travis-ci.org/r9y9/gossp.png?branch=master)](https://travis-ci.org/r9y9/gossp)
[![Coverage Status](https://coveralls.io/repos/r9y9/gossp/badge.svg?branch=master)](https://coveralls.io/r/r9y9/gossp?branch=master)
[![GoDoc](https://godoc.org/github.com/r9y9/gossp?status.svg)](https://godoc.org/github.com/r9y9/gossp)
[![License](http://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat)](LICENSE.md)


GOSSP is a Speech Signal Processing library for the Go language, which includes time-frequency analysis, fundamental frequency estimation, spectral envelope estimation and waveform generation filters, etc.

## Packages

- **cwt** - Continuous Wavelet Tranform (CWT) and inverse CWT (in develop).
- **[dct](http://godoc.org/github.com/r9y9/gossp/dct)** -  Discrete Cosine Transform (DCT) and Inverse DCT.
- **[dtw](http://godoc.org/github.com/r9y9/gossp/dtw)** -  Dynamic Time Warping (DTW)
- **[excite](http://godoc.org/github.com/r9y9/gossp/excite)** -  Excitation generation from fundamental frequency.
- **[f0](http://godoc.org/github.com/r9y9/gossp/f0)** -  Fundamental frequency (f0) estimatnion.
- **[io](http://godoc.org/github.com/r9y9/gossp/io)** -  Input/Output (in develop).
- **[mgcep](http://godoc.org/github.com/r9y9/gossp/mgcep)** - Mel-generalized cepstrum analysis for spectral envelope estimation.
- **[special](http://godoc.org/github.com/r9y9/gossp/special)** - Special functions analogy to scipy in python.
- **[stft](http://godoc.org/github.com/r9y9/gossp/stft)** - Short-Time Fourier Transform (STFT) and Inverse STFT.
- **[vocoder](http://godoc.org/github.com/r9y9/gossp/vocoder)** -  Speech waveform generation filters.
- **[window](http://godoc.org/github.com/r9y9/gossp/window)** -  Window functions.
- **[z](http://godoc.org/github.com/r9y9/gossp/z)** - Z-transform to analyze digital filters.

## Installation

```bash
go get github.com/r9y9/gossp
```

### Install SPTK

To use [SPTK](http://sp-tk.sourceforge.net/) with GOSSP, you need to install the modified version of SPTK as follows:

```bash
git clone https://github.com/r9y9/SPTK.git && cd SPTK
./waf configure && ./waf
sudo ./waf install
```

## Getting Started

### Short-Time Fourier Transform
![](examples/spectrogram.png)

```go
package main

import (
	"flag"
	"fmt"
	"github.com/r9y9/gossp"
	"github.com/r9y9/gossp/io"
	"github.com/r9y9/gossp/stft"
	"github.com/r9y9/gossp/window"
	"log"
	"math"
)

func main() {
	filename := flag.String("i", "input.wav", "Input filename")
	flag.Parse()

	w, werr := io.ReadWav(*filename)
	if werr != nil {
		log.Fatal(werr)
	}
	data := w.GetMonoData()

	s := &stft.STFT{
		FrameShift: int(float64(w.SampleRate) / 100.0), // 0.01 sec,
		FrameLen:   2048,
		Window:     window.CreateHanning(2048),
	}

	spectrogram, _ := gossp.SplitSpectrogram(s.STFT(data))
	PrintMatrixAsGnuplotFormat(spectrogram)
}

func PrintMatrixAsGnuplotFormat(matrix [][]float64) {
	fmt.Println("#", len(matrix[0]), len(matrix)/2)
	for i, vec := range matrix {
		for j, val := range vec[:1024] {
			fmt.Println(i, j, math.Log(val))
		}
		fmt.Println("")
	}
}
```

### Waveform Reconstruction using Inverse Short-Time Fourier Transform

![](examples/reconstructed_signal.png)

```go
package main

import (
	"flag"
	"fmt"
	"github.com/r9y9/gossp/io"
	"github.com/r9y9/gossp/stft"
	"github.com/r9y9/gossp/window"
	"log"
)

func main() {
	filename := flag.String("i", "input.wav", "Input filename")
	flag.Parse()

	w, werr := io.ReadWav(*filename)
	if werr != nil {
		log.Fatal(werr)
	}
	data := w.GetMonoData()

	s := &stft.STFT{
		FrameShift: int(float64(w.SampleRate) / 100.0), // 0.01 sec,
		FrameLen:   2048,
		Window:     window.CreateHanning(2048),
	}

	spectrogram := s.STFT(data)

	// do something on spectrogram

	reconstructed := s.ISTFT(spectrogram)

	for i := range data {
		// expect to be same
		fmt.Println(data[i], reconstructed[i])
	}
}
```

Fun with speech signal processing!

## Third Party Libraries

- [GO-DSP](https://github.com/mjibson/go-dsp)
- [MCEP Alpha Calculator](https://bitbucket.org/happyalu/mcep_alpha_calc/)
- [SPTK](http://sp-tk.sourceforge.net/)

## LICENSE

[MIT](./LICENSE)
