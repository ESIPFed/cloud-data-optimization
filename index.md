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
* [Pangeo benchmarking](https://github.com/pangeo-data/benchmarking) - Benchmarks the Pangeo platforms and its scaling

The following resources are not directly related to cloud data but may have relevant optimization efforts, background, and takeaways:

* [Making	earth	science	data	more	accessible: experience	with	chunking	and	compression](https://www.unidata.ucar.edu/presentations/Rew/ams-2013-rew-fixed.pdf) - Discussion of chunking decisions
* [SEED Data Manual](http://www.fdsn.org/pdf/SEEDManual_V2.4.pdf)
* [OAIS Reference Model](http://www.oais.info/)

## Factors Influencing Optimization Decisions

Performance optimization involves reducing the time to locate, retrieve, decode, and prepare data values for analysis.

To optimize for performance, the author should:

* Optimize data location, here defined as the amount of time it takes to determine which locatins in which files need to be read in order to fetched the desired data.  Different formats have different approaches to this problem, including:
  * Using sidecar or metadata files to describe byte offsets to different variables and spatiotemporal areas within the file
  * Storing byte offsets at predictable locations within the file that can be quickly read
  * Organizing files to allow direct seeks to the data without reading additional offset information
  * Splitting files across a filesystem (or other key/value storage like cloud object stores) so that desired portions of files can be referenced by name
* Optimize data retrieval time.  This involves firstly ensuring the latency and throughput of the system storing the data are as fast as feasible.  Assuming that is done, optimization of retrieval means ensuring that desired data can be fetched in the fewest number of reads while reading the smallest amount of extraneous data.  (Note that a client may nevertheless choose to parallelize reads for more efficient bandwidth usage or distributed processing.)  As these two concerns are highly use-case specific, this needs careful attention.
* Optimize data decoding.  Data are often compressed for both cost reasons and transfer performance.  Compression schemes and parameters need to balance cost, transfer performance, and decoding performance.
* Optimize preparation for analysis.  Once data values have been fetched, are they in the right format, projection, grid, units, etc?  Do they have the required quality information, metadata, etc required for valid analysis, and if not, what is required to fetch them?  Reducing the number and cost of these steps reduces both the development time to analysis and the raw computation time required for analysis.

Given the above, the answer for how to optimize data will always be "It depends."  The primary factors it depends on are:

* Likely access patterns, which are usually driven by likely analysis needs.  Data should be arranged such that desired bytes can be located and read in as few requests as possible, while reading as little unnecessary data as possible.
* Latency and throughput of the data store.  Object stores tends to be relatively high latency and high throughput.  This means that reading extraneous bytes can often be worthwhile if it means making fewer requests.  A number to measure and monitor is the number of bytes that could be transfered during the latency period of a request.  Below this number, it tends to be better to read extraneous bytes, while above it it tends to be better to perform a separate request.
* End user location and network performance.  Similar to the previous item, but on the user side, in-cloud access can have wildly different performance (and cost) characteristics than out-of-cloud access, which themselves can vary greatly.
* Analysis needs.  What are users trying to do?  What software is readily available to them?  What projections, grids (if any), units, etc do they need most?  Which data variables are likely to be needed together?  What is required to make valid use of science data?  Reducing time to analysis means considering these factors, though they are largely out of scope of this document.

While the above optimizes for performance, these concerns need to be balanced against organizational requirements, regulatory requirements, and other factors such as creation, hosting, and transfer cost.

### Optimization Practices

#### Chunking 

For many file formats, including HDF, NetCDF, and Zarr, chunks constitute the smallest unit of data within a file that can be read at once.  Reading a chunk of data incurs a latency cost, so chunks ought not be too small or performance will suffer.  Reading a chunk of data also incurs a data transfer and decoding cost, so chunks ought not be too large or software may need to transfer and process many extraneous bytes for a particular use case and may also need to keep bytes in slower memory locations, compounding issues.  There is a fundamental tension, where use cases that need a small amount of data from a dimension, such as a time series at a point in space, suffer from a large chunk size in that dimension, while use cases that need a large amount of data from that dimension, such as a spatial analysis at a point in time, suffer from a small chunk size in that dimension.

The optimal chunk shape varies based on expected use cases, but it also varies with the latency and throughput of the data store.  For data in cloud storage, the latter characteristics can be dramatically different from data stored on a local disk.  Chunk size for cloud stores therefore needs careful consideration and cannot easily rely on non-cloud rules of thumb.

##### Chunk Size 

A chunk size should be selected that is large in order to reduce the number of tasks that parallel schedulers like Dask have to think about (which affects overhead) but also small enough so that many of them can fit in memory at once. The Pangeo project has been recommending a chunk size of about 100MB, which originated from the [Dask Best Practices](https://docs.dask.org/en/latest/array-best-practices.html#select-a-good-chunk-size).  The [Zarr Tutorial](https://zarr.readthedocs.io/en/v2.0.0/tutorial.html#chunk-size-and-shape) recommends a chunk size of at least 1MB.   The [Amazon S3 Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/dev/optimizing-performance-guidelines.html#optimizing-performance-guidelines-get-range) says the typical size for byte-range requests is 8-16MB. It would seem that chunk sizes on the order of 10MB or 100MB are most optimal for Cloud usage.

### Antipatterns

A community survey of the ESIP Cloud Computing Cluster noted the following antipatterns in cloud data:

* Large granule sizes with no means to subset those granules, requiring full-file transfers
* Storing data uncompressed, which can increase peformance but double cost
* Lift and shift, which fails to make use of the cloud's elasticity and highly scalable storage while also ignoring changes in access performance
* Large central file mounts, which create scalability and performance issues
* Persistent processes, especially inelastic one.  Scientific workloads tend to have high variability in demand and require attention to scaling to provide burst performance without inordinate cost
* Small chunk sizes, which create significant network chatter
* Web-based portals instead of direct programmatic data access, which limits automated activity and scalability while returning too many results
* Failing to cache for repeat access, both client- and server-side, and particularly in service outputs which are expensive to produce and need idempotency
* Putting workflows on the client, which creates significant back-and-forth nework traffic
* Hierarchical data tree walks, which tend to be slow and prone to runtime errors

## Moving forward

Please contribute to this document if you have input.  We welcome pull requests.

Additionally, the community has noted the following specific needs for input or experimentation:

1. Identifying commonalities across communities, organizations, and source data formats
2. Performance analysis on a variety of data organizations, analyses and data structure types.
3. Chunking and compression options in the context of scalable data access to model output
4. Data on how optimization decisions vary between different access clients like remote users, dask, and spark clusters.
