# Ray Tracing Gems

##使用DXR和其他的APi进行高质量并且实时的渲染

主编：Eric Haines ，Tomas Akenine-Moller


分章主编：
Alexander Keller
Morgan McGuire
Jacob Munkberg
Matt Pharr
Peter Shirley
Ingo Wald
Chris Wyman

# Contents

## Part I: Ray Tracing Basics 5
[序言](Preface.md)
[前言](Foreword.md)
1 Ray Tracing Terminology 7
1.1 Historical Notes 
1.2 Definitions 8
2 What is a Ray? 13
2.1 Mathematical Description of a Ray 13
2.2 Ray Intervals 14
2.3 Rays in DXR 15
2.4 Conclusion 16
3 Introduction to DirectX Raytracing 17
3.1 Introduction 17
3.2 Overview 18
3.3 Getting Started18
3.4 The DirectX Raytracing Pipeline 19
3.5 New HLSL Support for DirectX Raytracing 20
3.6 A Simple HLSL Ray Tracing Example 23
3.7 Overview of Host Initialization for DirectX Raytracing 
3.8 Basic DXR Initialization and Setup 25
3.9 Ray Tracing Pipeline State Objects 30
3.10 Shader Tables 33
3.11 Dispatching Rays35
3.12 Digging Deeper and Additional Resources 36
3.13 Conclusion 37
4 A Planetarium Dome Master Camera 39
4.1 Introduction 39
4.2 Methods 39
4.3 Planetarium Dome Master Projection Sample Code 46
5 Computing Minima and Maxima of Subarrays 49
5.1 Motivation 49
5.2 Naive Full Table Lookup 50
5.3 The Sparse Table Method 50
5.4 The (Recursive) Range Tree Method 52
5.5 Iterative Range Tree Queries 52
5.6 Results 55
5.7 Summary 55
## Part II: Intersections and Efficiency 
6 A Fast and Robust Method for Avoiding
Self-Intersection 63
6.1 Introduction 63
6.2 Method 63
6.3 Conclusion 69
7 Precision Improvements for Ray/Sphere
Intersection 71
7.1 Basic Ray/Sphere Intersection 71
7.2 Floating-Point Precision Considerations 72
7.3 Related Resources76
8 Cool Patches: A Geometric Approach to Ray/Bilinear Patch Intersections 79
8.1 Introduction and Prior Art 79
8.2 GARP Details 83
8.3 Discussion of Results85
8.4 Code 88
9 Multi-Hit Ray Tracing in DXR 93
9.1 Introduction 93
9.2 Implementation95
9.3 Results 99
9.4 Conclusions 105
10 A Simple Load-Balancing Scheme with High
Scaling Efficiency 107
10.1 Introduction 107
10.2 Requirements 107
10.3 Load Balancing 108
10.4 Results 111
## Part III: Reflections, Refractions, and Shadows 
11 Automatic Handling of Materials in Nested
Volumes 119
11.1 Modeling Volumes 119
11.2 Algorithm 121
11.3 Limitations 125
12 A Microfacet-Based Shadowing Function to Solve
the Bump Terminator Problem 127
12.1 Introduction 127
12.2 Previous Work 128
12.3 Method 128
12.4 Results 133
13 Ray Traced Shadows: Maintaining Real-Time
Frame Rates 137
13.1 Introduction 137
13.2 Related Work 138
13.3 Ray Traced Shadows 139
13.4 Adaptive Sampling 141
13.5 Implementation 147
13.6 Results 150
13.7 Conclusion and Future Work153
14 Ray-Guided Volumetric Water Caustics in
Single Scattering Media with DXR 157
14.1 Introduction 157
14.2 Volumetric Lighting and Refracted Light 159
14.3 Algorithm 162
14.4 Implementation Details 167
14.5 Results 168
14.6 Future Work 170
14.7 Demo 170
## Part IV: Sampling 175
15 On the Importance of Sampling 
15.1 Introduction 177
15.2 Example: Ambient Occlusion 178
15.3 Understanding Variance 182
15.4 Direct Illumination 184
15.5 Conclusion 186
16 Sampling Transformations Zoo 189
16.1 The Mechanics of Sampling 189
16.2 Introduction to Distributions189
16.3 One-Dimensional Distributions 191
16.4 Two-Dimensional Distributions 195
16.5 Uniformly Sampling Surfaces198
16.6 Sampling Directions202
16.7 Volume Scattering 205
16.8 Adding to the Zoo Collection 206
17 Ignoring the Inconvenient When Tracing Rays 209
17.1 Introduction 209
17.2 Motivation 209
17.3 Clamping 211
17.4 Path Regularization 212
17.5 Conclusion 213
18 Importance Sampling of Many Lights on the GPU 215
18.1 Introduction 215
18.2 Review of Previous Algorithms 216
18.3 Foundations 219
18.4 Algorithm 223
18.5 Results 228
18.6 Conclusion 234
## Part V: Denoising and Filtering 
19 Cinematic Rendering in UE4 with Real-Time
Ray Tracing and Denoising 245
19.1 Introduction 245
19.2 Integrating Ray Tracing in Unreal Engine 4246
19.3 Real-Time Ray Tracing and Denoising 254
19.4 Conclusions 268
20 Texture Level of Detail Strategies for Real-Time Ray Tracing 271
20.1 Introduction 271
20.2 Background 272
20.3 Texture Level of Detail Algorithms273
20.4 Implementation 283
20.5 Comparison and Results 284
20.6 Code 288
21 Simple Environment Map Filtering Using Ray
Cones and Ray Differentials 293
21.1 Introduction 293
21.2 Ray Cones 293
21.3 Ray Differentials 294
21.4 Results 295
22 Improving Temporal Antialiasing with Adaptive Ray Tracing 297
22.1 Introduction 297
22.2 Previous Temporal Antialiasing 299
22.3 A New Algorithm 299
22.4 Early Results 306
22.5 Limitations 308
22.6 The Future of Real-Time Ray Traced Antialiasing 309
22.7 Conclusion 310
## Part VI: Hybrid Approaches and Systems 
23 Interactive Light Map and Irradiance Volume
Preview in Frostbite 319
23.1 Introduction 319
23.2 GI Solver Pipeline 320
23.3 Acceleration Techniques 334
23.4 Live Update 338
23.5 Performance and Hardware 339
23.6 Conclusion 345
24 Real-Time Global Illumination with Photon Mapping 347
24.1 Introduction 348
24.2 Photon Tracing 349
24.3 Screen-Space Irradiance Estimation 354
24.4 Filtering 360
24.5 Results 365
24.6 Future Work 368
25 Hybrid Rendering for Real-Time Ray Tracing 371
25.1 Hybrid Rendering Pipeline Overview 371
25.2 Pipeline Breakdown 373
25.3 Performance 397
25.4 Future 399
25.5 Code 399
26 Deferred Hybrid Path Tracing 405
26.1 Overview 405
26.2 Hybrid Approach 405
26.3 BVH Traversal 408
26.4 Diffuse Light Transport 410
26.5 Specular Light Transport 413
26.6 Transparency 415
26.7 Performance 416
27 Interactive Ray Tracing Techniques for High-Fidelity Scientific Visualization 421
27.1 Introduction 421
27.2 Challenges Associated with Ray Tracing Large Scenes 
27.3 Visualization Methods 427
27.4 Closing Thoughts 438
## Part VII: Global Illumination 
28 Ray Tracing Inhomogeneous Volumes 447
28.1 Light Transport in Volumes 447
28.2 Woodcock Tracking448
28.3 Example: A Simple Volume Path Tracer449
28.4 Further Reading 453
29 Efficient Particle Volume Splatting in a RayTracer 
29.1 Motivation 457
29.2 Algorithm 458
29.3 Implementation 459
29.4 Results 462
29.5 Summary 462
30 Caustics Using Screen-Space Photon Mapping 465
30.1 Introduction 465
30.2 Overview 466
30.3 Implementation 467
30.4 Results 471
30.5 Code 473
31 Variance Reduction via Footprint Estimation
in the Presence of Path Reuse 475
31.1 Introduction 475
31.2 Why Assuming Full Reuse Causes a Broken MIS Weight 
31.3 The Effective Reuse Factor 477
31.4 Implementation Impacts 482
31.5 Results 482
32 Accurate Real-Time Specular Reflections with Radiance Caching 
32.1 Introduction 488
32.2 Previous Work 489
32.3 Algorithm 490
32.4 Spatiotemporal Filtering 500
32.5 Results 508
32.6 Conclusion 512
32.7 Future Work 512
