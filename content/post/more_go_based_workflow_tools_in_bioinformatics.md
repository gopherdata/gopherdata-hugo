+++
title = "More Go-based Workflow Tools in Bioinformatics"
date = "2017-11-02T23:10:02+01:00"

math = false
highlight = true
tags = ["workflows", "bioinformatics"]

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""
+++

It is an exciting time for Go as a data science language and for the
[\#gopherdata](http://gopherdata.io/) movement. The 
[ecosystem of tools](https://github.com/gopherdata/resources/blob/master/tooling/README.md)
is constantly improving, with both general purpose tools (e.g., for data frames and
statistical analyses) and more specialized ones (e.g., for neural networks and
graph-based algorithms) popping up every day.

Recent weeks have been particularly exciting for those involved in the
bioinformatics field. In addition to generic libraries for bioinformatics such
as [bíogo](https://github.com/biogo/biogo), which was recently reviewed in quite some detail in a two blog posts
([part I](https://medium.com/@boti_ka/a-gentle-introduction-to-b%C3%ADogo-part-i-65dbd40e31d4)
and [part II](https://medium.com/@boti_ka/a-gentle-introduction-to-bíogo-part-ii-1f0df1cf72f0)),
the ecosystem of scientific workflow tools focusing on or being used in
bioinformatics is also growing: Last week, another Go-based workflow
orchestration tool, [Reflow](https://github.com/grailbio/reflow), was released
as open source, by life science startup [Grail Inc](https://grail.com/). 

Reflow brings a number of interesting features to the
table including: (i)&nbsp;comprehensive and very easy-to-use Amazon Web
Services (AWS) integration, (ii)&nbsp;memoization features based on S3
storage, and (iii)&nbsp;ability to run the same, docker-based workflow on your
local computer or on AWS.

Because we are users of and contributors to two other existing Go-based
workflow projects ([Pachyderm](http://pachyderm.io/) and
[SciPipe](http://scipipe.org/)), we thought that a brief comparison of the
different approaches would be useful. In particular, we hope that this summary
helps highlight differences between the tools, guide users to appropriate
workflow and orchestration tools, and/or provide a jumping-off-point for
contributions and experimentation

Workflow Tools in Bio- and Cheminformatics
----------------------------------------------------

Before we continue, lets step back and add a few words about what workflow
tools are and why they are relevant in Bioinformatics (or Science in general),
for anyone new to the term. 

[Scientific workflow tools](https://en.wikipedia.org/wiki/Scientific_workflow_system) are tools or
systems designed to help coordinate computations when the number of computation
steps is very large and/or the steps have complex dependencies between their
input and output data. This is common in many scientific fields, but it is
common in bioinformatics in particular, because of the vast plethora of tools
built to analyze the extremely heterogeneous data types describing biological
organisms, from molecules to cells to organs to full body physiology.

Common tasks in bioinformatics necessitating the use of workflow tools include
(DNA) sequence alignment, (gene) variant calling, RNA quantification, and more.
In the related field of cheminformatics, workflow tools are often used to build
predictive models relating the chemical structure of drug compounds to some
measurable property of the compound, in what is commonly called Quantitative
Structure Activity Relationship (QSAR). Examples of such properties are the
level of binding to protein targets in the body, or various chemical properties
such as solubility, that might affect its potential for update in the body.

The Go bioinformatics workflow tooling ecosystem
------------------------------------------------

![Gopher thinking about workflows, surrounded by workflow tool logos](/img/wftools/gopher_thinking_workflows.png)

The main Go-based workflow tools that have targeted bioinformatics workflows
(or have been used to implement bioinformatics workflows) are:

**[Pachyderm](http://pachyderm.io/)** - This platform allows users to build
scalable, containerized data pipelines using any language or framework, while
also getting the right data to the right code as both data and code change over
time. Individual pipeline stages are defined by docker images and can be
individually parallelized across large data sets (by leveraging
[Kubernetes](https://kubernetes.io/) under the hood).

**[SciPipe](http://scipipe.org/)** - This tool is much smaller than something
like Pachyderm and is focused on highly dynamic workflow constructs in
which on-line scheduling is key. SciPipe also focuses, thus far, on workflows
that run on HPC clusters and local computers.  Although, it can be integrated
in a more comprehensive framework, such as Pachyderm or Reflow.

**[AWE](https://github.com/MG-RAST/AWE)** - The AWE framework is a
comprehensive bioinformatics system targeting the cloud. It reportedly
comes with multi-cloud support and, as it seems, HPC support, and AWE allows
users to leverage the [Common Workflow Language (CWL)](http://www.commonwl.org/). As
with Reflow, we don’t have hands-on experience with AWE, so our comments
are limited to what we’ve read in the documentation and examples.

Comparing Reflow and Pachyderm
------------------------------

Reflow and Pachyderm seems to be closest at first glance. Both frameworks
utilize containers as the main unit of data processing. However, there are some
differences in both workflow orchestration and data storage/management that we
will stress below.

### Orchestration

While Reflow implements custom “runners” for each target environment (AWS and
your local machine currently), Pachyderm leverages Kubernetes under-the-hood
for container orchestration. This use of Kubernetes allows Pachyderm to
be maximally vendor-agnostic, as Kubernetes is widely deployed in all the
major clouds and on-premise. Reflow authors are reportedly planning to
support other cloud providers, like GCP, in the near future though, but
this effort seems to require writing custom runner code per target
environment/cloud.

The use of Kubernetes also allows Pachyderm to automatically bring certain
orchestration functionality into a context that has otherwise
propelled Kubernetes to become industry leader in orchestration. More
specifically, Pachyderm pipelines are automatically “self-healing” in that they are
rescheduled and restarted when cluster nodes or jobs fail, and Pachyderm is
able to optimally schedule work across high-value nodes (e.g., high-CPU or GPU
nodes) using methods such as auto-scaling. Reflow appears to use it’s own
logic to match workloads with available instance types.

Finally, Pachyderm and Reflow have differences in the way workflows are
actually defined. Pachyderm leverages the established JSON/YAML format that is
common in the Kubernetes community, and Reflow implements what seems to be its
own readable, Go-inspired domain specific language (DSL).

### Data Storage/Management

For the data that is to be processed in workflow stages, Reflow provides an
interesting memoization mechanism based on S3, with automatic re-runs triggered
by updates to data. Similarly, Pachyderm provides a familiar, git-inspired
versioned data store, where new versions of data in the versioned data store
are also used to automatically trigger relevant parts of the workflow.
Pachyderm’s data store can be backed by any of the popular vendor-specific
object stores (GCS in GCP, S3 in AWS, or Blob Storage in Azure) or any other
object store with an S3-compatible API (e.g., an open source option like
Minio).

In terms of metadata associated with jobs, versions of data, etc., Reflow
appears to use a Reflow-specific cache based on S3 and DynamoDB. In
contrast, Pachyderm utilizes etcd, a distributed key-value store, for metadata.

Lastly with regard to data sharding and parallelization, Pachyderm seems to go
further than Reflow. While Reflow does parallelize tools running on separate
data sets, Pachyderm also provides automatic parallelization of tools accessing
“datums” that may correspond to the same data set or even the same
file. Pachyderm automatically distributes the processing of these datums to
containers running in parallel (pods in Kubernetes) and gathers all of the
results back into a single logical collection of data. Pachyderm thus provides
what other frameworks like Hadoop and Spark are promising, but without the need
to replace legacy code and tools with code written in the MapReduce-style or
explicitly implement parallelism and data sharding in code.

With these notes in mind, we think Reflow and Pachyderm are addressing slightly
different user needs. While Reflow seems to be an excellent choice for a quick
setup on AWS or a local docker-based server, we think Pachyderm will generally
provide more vendor agnosticity, better parallelization, and valuable
optimizations and updates that come out of the growing
Kubernetes community. Finally, we think that Pachyderm provides a stronger
foundation for rigorous and manageable data science with its unified data
versioning system, which can help data scientists and engineers better
understand data, collaborate, perform tests, share workflows, and so on.

How does SciPipe compare?
-------------------------------------------------

SciPipe is, in this context, more of an apples-to-oranges comparison to
Pachyderm, Reflow or AWE. While Reflow and Pachyderm provides an integrated
tool encapsulation solution based on containers, SciPipe (in its current
form) is primarily focused on managing command-line driven workflows on local
computers or HPC clusters with a shared file system, where containers might not
be an option. However, there are also other relevant differences related to
complexity of the tools, workflow implementation, and deployment/integration:

### Complexity

SciPipe is a much smaller tool in many ways. For example, a very simple
count of LOC for the different frameworks shows that SciPipe is implemented with
more than an order of magnitude less lines of code than the other tools:

```bash
$ cd $GOPATH/src/github.com/grailbio/reflow
$ find | grep "\.go" | grep -vP “(vendor|examples|_test)” | xargs cat | grep -vP "^\/\/" | sed '/^\s*$/d' | wc -l
26371
```

```bash
$ cd $GOPATH/src/github.com/pachyderm/pachyderm/src
$ find | grep "\.go" | grep -vP “(vendor|examples|_test)” | xargs cat | grep -vP "^\/\/" | sed '/^\s*$/d' | wc -l
25778
```

```bash
$ cd $GOPATH/src/github.com/MG-RAST/AWE
$ find | grep "\.go" | grep -vP "(vendor|examples|_test)" | xargs cat | grep -vP "^\/\/" | sed '/^\s*$/d' | wc -l
24485
```

```bash
$ cd $GOPATH/src/github.com/scipipe/scipipe
$ find | grep "\.go" | grep -vP “(vendor|examples|_test)” | xargs cat | grep -vP "^\/\/" | sed '/^\s*$/d' | wc -l
1699
```

### Workflow Implementation

Further, SciPipe was designed to primarily support highly dynamic workflow
constructs, where dynamic/on-line scheduling is needed. These workflows include
scenarios in which you are continuously chunking up and computing a dataset of
unknown size or parametrizing parts of the workflow with parameter values
extracted in an earlier part of the workflow. An example of the former would
be lazily processing data extracted from a database without saving the
temporary output to disk. An example of the latter would be doing a parameter
optimization to select, e.g., good gamma and cost values for libSVM before
actually training the model with the obtained parameters.

Also, where Pachyderm and Reflow provide manifest formats or DSLs for writing
workflows, SciPipe lets you write workflows directly in Go. SciPipe is
thus consumed as a programming-library rather than a framework.
This feature might scare off some users intimidated by Go’s relative
verboseness compared to specialized DSLs, but it also allows users to leverage
extremely powerful existing tooling and editor support for Go.

### Deployment/Integration

What is perhaps most interesting in the context of this comparison is the fact
that SciPipe workflows can be compiled to small static binaries. This
compilation makes it very easy to package up smaller SciPipe workflows in
individual containers and integrate them into tools like Pachyderm or Reflow or
other services. We thus imagine that SciPipe could be a complement to Pachyderm
or Reflow when highly dynamic workflow constructs are needed, which
may be a challenge to implement in the manifests and DSLs of Pachyderm or
Reflow.

Reflow and AWE?
---------------------------------------

We know the least about AWE at this point, so we don’t want to venture too far
into a detailed comparison with Reflow (which is also new to us). However,
based on our reading of online materials, we can note that they seem to share
the focus on bioinformatics and cloud support. AWE additionally supports Common
Workflow Language and HPC clusters. We expect there to be some differences in
terms of storage, because AWE ships with its own storage solution and isn’t
based on a cloud offering like S3. Past that, we will leave it to the authors
of AWE and Reflow or the community to provide more comprehensive comparisons.

In Summary
-----------------------

In summary, we think that it is extremely exciting and reassuring to see
continued innovation in the Go Data Science ecosystem, and we are excited to
see more and more data gophers and projects join the community.  We also hope
this little overview will help users navigate the growing ecosystem of workflow
tools by highlighting some of their inherent differences.

----

[Samuel Lampa](https://twitter.com/smllmp), PhD Student at [Uppsala University](http://pharmb.io)<br>

[Jon Ander Novella](https://www.linkedin.com/in/jon-ander-novella/), Research Assistant at [Uppsala University](http://pharmb.io)<br>

[Daniel Whitenack](https://twitter.com/dwhitena), Data Scientist and Lead Developer Advocate at [Pachyderm Inc.](http://pachyderm.io)
