# Cloud Hosted Data Optimization Knowledge Base
## ESIP Cloud Computing Cluster

This repository captures optimization practices for data in the cloud.

## Why Optimize Data for Cloud Access?

Increasingly, organizations with large data holdings are turning to cloud-hosted
storage to improve capacity, scalability, and access to computing resources near
data.  These data are often stored in object stores or other storage that offers
horizontally-scalable access over a network.  This demands attention from data
providers seeking to maximize the value of data, as the performance
characteristics of cloud-hosted stores can differ greatly from their
predecessors.  Further, the cost profile of cloud hosting differs from that of
on premises hosting, such that unnecessary data requests, data transfer, and
compute time all tend to directly contribute to the overall cost to host data.

In short, well-optimized data in a cloud environment are typically less
expensive, more usable, and have faster typical access times than poorly optimized
data.

## Resources for Cloud Data Optimization

* [Task 51 - Cloud-Optimized Format
  Study](https://ntrs.nasa.gov/search.jsp?R=20200001178) - A NASA study on the
  performance characteristics of several formats in cloud environments.
* [Landsat Cloud Optimized GeoTIFF Data Format Control
  Book](https://www.usgs.gov/media/files/landsat-cloud-optimized-geotiff-data-format-control-book)  -
  Details on format selections and packaging for Landsat Collection 2.
* [Highly Scalable Data Service (HSDS)](https://github.com/HDFGroup/hsds) - A REST-based web service for HDF5 data stores, built by The HDF Group, with
  several optimizations for data access in network environments.
* [Performance Guidelines for Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/optimizing-performance-guidelines.html) - Amazon suggestions to optimize performance on S3.

## Factors Influencing Optimization Decisions

_(WIP: analysis patterns, data size, tradeoffs in cost /
performance.  See "Background" section of Google Doc.)_

### Optimization Practices

_(WIP: chunk sizes, sidecar files, NetCDF -> Zarr work)_

### Chunking 

When defining chunks, there are many options to specify.  What are the considerations for chunk size, shape, compression and filter options?  

**Chunk Size**: 

A chunk size should be selected that is large in order to reduce the number of tasks that parallel schedulers like Dask have to think about (which affects overhead) but also small enough so that many of them can fit in memory at once. The Pangeo project has been recommending a chunk size of about 100MB, which originated from the [Dask Best Practices](https://docs.dask.org/en/latest/array-best-practices.html#select-a-good-chunk-size).  The [Zarr Tutorial](https://zarr.readthedocs.io/en/v2.0.0/tutorial.html#chunk-size-and-shape) recommends a chunk size of at least 1MB.   The [Amazon S3 Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/dev/optimizing-performance-guidelines.html#optimizing-performance-guidelines-get-range) says the typical size for byte-range requests is 8-16MB. It would seem that chunk sizes on the order of 10MB or 100MB are most optimal for Cloud usage.
 


### Antipatterns

_(WIP: No lift and shift, avoiding large granules without a means to subset)_

### Cost and Compliance Considerations

_(WIP: egress controls, export controls, multi-tenant systems, etc)_

