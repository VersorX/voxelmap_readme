# VoxelMap - The CUDA Voxel Mapping Library

VoxelMap is a high-performance occupancy grid mapping library for processing large-scale depth measurements in real-time, using CUDA-enabled GPUs.

# Table of Contents  
* [Overview](#Overview)
* [Features](#Features)
    * [Occupancy grid mapping](#Occupancy-grid-mapping)  
    * [Serialization](#Serialization)
    * [Visualization](#Visualization)
* [Requirements](#Requirements)
    * [Build dependencies](#Build-dependencies)
    * [Run-time dependencies](#Run-time-dependencies)
* [Benchmarks](#Benchmarks)

# Overview

Mobile robots need a good representation of the environment to move around collision-free. It is potentially very costly to update a global map with high-frequency sensor measurements. VoxelMap uses hardware accelerated parallelism and concurrent data structures to efficiently process large-scale measurement data in real-time.

# Features

## Occupancy grid mapping

VoxelMap maintains a probabilistic occupancy grid to incorporate noisy and uncertain measurement updates. The map is stored as a concurrent voxel-based tree data structure for highly efficient parallel updates.

## Serialization



## Visualization

# Requirements

## Build dependencies

Libraries required at compile-time:
* Eigen3
* CUDA

## Run-time dependencies

Run-time dependencies are limited to:
* libcudart.so

# Benchmarks

## Synthetic data

![simple_benchmark](gifs/simple_benchmark.gif)

To get a rough estimate of the performance of VoxelMap, the average time to incorporate a new scan is measured. The scan is created by setting each distance measurement equal some assumed average trace length. Each scan is identical, but the sensor origin is shifted 1m along the x-axis for each insert operation. This is repeated 1000 times, or until 10 minutes has passed, and the average update time is computed. This procedure will by no means yield realitic maps, but it will give an indication of runtime cost for a given sensor setup. Update times for Octomap, with the exact same input, is included for comparison.

The benchmark is run on NVIDIA's Jetson AGX Xavier, with a voxel size of 0.5m. The tables below show the average update time, given an assumed average trace length.


### Sensor A
* 64 channels, 45&deg; VFOV
* 1024 sectors, 360&deg; HFOV

Library | 40m average trace | 60m average trace | 80m average trace
--- | --- | --- | ---
VoxelMap | 7.8 ms | 13.6 ms | 19.8 ms
OctoMap | 2.0 s | 6.7 s | 13.8 s

### Sensor B
* 128 channels, 45&deg; VFOV
* 2048 sectors, 360&deg; HFOV

Library | 40m average trace | 60m average trace | 80m average trace
--- | --- | --- | ---
VoxelMap | 27.2 ms | 44.6 ms | 65.0 ms
OctoMap | 3.2 s | 9.0 s | 21.5 s



## Real data
