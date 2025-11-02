---
layout: default
title: Home
---

<script src="https://ncase.me/nutshell/nutshell.js"></script>
<script>
  Nutshell.setOptions({
    startOnLoad: true,
  });
</script>

# : Image Encryption

## : Introduction
	Images like any other data format can very easily be encrypted using standard algorithms like RSA or AES but those result in a meaningless blob that cant be used anywhere. Sometimes it's worth keeping some properties to allow for file uploads to share it with others in places where images are only allowed.

## : Basic Encryption
	This step is fairly easy to implement, one can easily take a block of a few pixels and encrypt them and interpret the resulting data also as pixels. This allows one to upload the encrypted image on social media or chat apps that restrict file uploads to images. However some apps do additional processing like scaling the image and the new algorithm for encryption doesn't work anymore. When the image is scaled the algorithm usually involves some interpolation so the resulting pixels are different from the starting pixels and the decryption breaks
	//TODO: add image for this

	The knee-jerk reaction to trying to work with encrypted data is [:Homomorphic Encryption](#HomomorphicEncryption). However there are a [:few specific issues](#IssuesWithHomomorphicEncryption) that make this not optimal.

## :x Homomorphic Encryption
	Homomorphic Encryption is a scheme where operations can be perfomed on the encrypted data which corresponds to an operation on the initial data. This can be written as encrypt(f(a)) = g(encrypt(a)) or in the binary operation case encrypt(f(a,b)) = g(encrypt(a),encrypt(b))

## :x Issues with Homomorphic Encryption
	In Homomorphic Encryption there is no control over the algorithm used on the encrypted image, this is both in terms of the actual logic of the algorithm and also that it wont involve the required operators that map from the unencrypted domain to the encrypted domain. Secondly the calculations are usually performed in floating point and converted to integers at the end which doesn't play well with most Homomorphic Encryption schemes

## :x The Problem
	Before we move on with the potentials solutions we have to define the problem to be solved first:
	  - Encryption must convert images to images.
	  - Images must be in a commonly used format preferably the same format as the input image. [:Why?](#whyACommonFormat)
	  - Encrypted image must be the same size as the initial image. [:Why?](#whySameSize)
	  - When the encrypted image is scaled down and decrypted the result should be a scaled version of the initial image, i.e. Decrypt(Scale(Encrypt(Img))) = Decrypt(Enc(Scale(Img))) = Scale(Img), The above property should hold for most common scaling algorithms.
	  - Encryption must be "backed" by a standard encryption algorithm to ensure security.
	  - Encryption may be lossy as long as the image is mostly preserved.

### :x Why a Common Format
	  This is so that the image can be rendered and scaled by the application

### :x Why Same Size
      This is so that the user need not waste extra bandwidth for downloading large images just because they are encrypted

## Frequency Domain
	Now that we have specified the constraints the most obvious solution is to work with the image in the frequency domain with a [:Direct Cosine Transform](#DCT).	This is because a scaling operation is [:analogous to a crop](#DCTAndScaling) of the top left part of the DCT of the corresponding size.

### :x DCT
	DCT is an operation that converts an image to a same shaped matrix of frequencies with the lower frequencies being at the top left and the higher frequencies at the bottom right. The coefficients are real valued unlike a fourier transform and they represent a sum of cosines. It is an invertable operation and theoretically no data is lost however floating point errors will creep in.

### :x DCT and Scaling
	Scaling process deletes high frequency data and leaves the low frequency data. When an image is scaled usually the high quality detail is smoothed away leading to the deletion of the high frequency data but the low frequency data which tells more about the image overall is preserved. In other words scaling is basically a resampling of the signal which would lead to the high frequency region to get cutoff.

## The Naïve Encryption
	The naïve way to encrypt would be to use AES. This is by taking 2 coefficients at a time and encrypting them. [:Why?](#why2Coefficients) the resulting coefficients can be converted to an image through an inverse DCT to get the final encrypted image
	This method has an issue the resulting image will have amplitudes which aren't bounded, however we need the image amplitudes to be between 0 and 255

### Why 2 Coefficients
	Each coefficent will be represented as a double precision float(float64), since AES takes 128 bits as an input two of them together can form the input and the encrypted result of 128 bits can be split back into two floats, while decrypting if any pair is split due to the cropping it has to be neglected but the rest of the image can be decrypted resulting in a slightly smaller image.
