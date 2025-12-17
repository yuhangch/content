---
id: dON
title: WPS 2.0 Translation
pubDate: 2019-11-16T02:53:04.000Z
isDraft: true
tags:
  - spatial
  - ogc
categories:
  - translation
---

## WPS

#### Providing a capability discovery interface

##### Every WPS service should be self-contained and provide an initial endpoint that can be used by WPS clients to determine the service capabilities

-   Initial endpoint (HTTP URI).
-   The service should provide a systematic mechanism for discovering its capabilities.
-   The discovery mechanism should be logical and inferable.

#### Abstract process model

The abstract process model provides many dimensions and degrees of freedom for process description.

-   A process must provide a unique identifier to distinguish it from processes in other process groups.
-   The identifier of a process should be a string or, preferably, a URI.
-   A process may have an arbitrary number of inputs (0 or more).
-   Each input of a process should have an identifier to distinguish it from other inputs.
-   The identifier of an input should be a string.
-   The parameters of process inputs should be nested.
-   Nested inputs should have identifiers that differ from those of other child nodes.
-   A process should have one or more outputs.
-   Each output of a process should have a unique identifier to distinguish it from other outputs in the process.
-   The identifier of an output should be a string.
-   The cardinality of each process output is 1.
-   Process outputs should be nested.
-   The identifier of an output should be different from that of its parent node.
-   All inputs and outputs that are not parent nodes should have well-defined data formats.
-   If input/output require encoding tables, they should use the definitions listed in **Table 2**.

#### Job control

Execution capabilities allow WPS clients to instantiate and execute jobs and are an important aspect of job control. In addition, the ability to cancel and delete a job during its execution is very meaningful for freeing server resources in time-critical execution.

-   The service should assign a unique identifier to each job.
-   The service should be able to return an exception if a client attempts to use an invalid job identifier.
-   The service should provide the capability to execute a process and create a new job for a process that can be executed, enabling clients to run the processes defined in the execution capabilities.
-   The service should provide the capability to cancel a previously submitted job. This allows clients to indicate that they are no longer interested in this job or its results, enabling the server to release any associated computing resources as far as possible.

#### Process execution

Execution of processes on a WPS service should support both synchronous and asynchronous modes. Synchronous execution is suitable for operations that can be completed in a short time, while asynchronous execution is more suitable for operations that require a long time to complete.

In the synchronous case, a WPS client submits an Execute request to a WPS server and remains waiting for feedback until the job has finished executing and the result is returned. This requires maintaining a persistent connection between client and server.

In the asynchronous case, the client sends an Execute request to the WPS server and immediately receives a response containing status information. This information confirms that the request has been received and accepted by the server, that the job is being processed, and that it will complete at some time in the future. The status information should also include the identifier of the process so that the client can later check whether the job has completed. In addition, the status information should contain the location of the result, such as a URL at which the processing result can be accessed after the job finishes.

-   The service should allow clients to specify which process is to be executed.
-   The service should return an exception when a client attempts to execute a process that cannot be executed.
-   For successful execution, the server should send a response to the client containing output data or a reference (index) to the data.
-   Execution results should have an expiration time, after which the output data will no longer be available.
-   For failed operations, the service should return an exception to the client.
-   The service should allow clients to specify the input data used for process execution.
-   An exception should be returned when a client specifies invalid input data for process execution.
-   The service should allow clients to define a desired data exchange format for output results, selected from the data formats the process indicates it can provide.
-   When a client specifies an unsupported data exchange format, the service should return an exception.
-   The service should indicate whether it supports synchronous execution, asynchronous execution, or both.
-   For each offered process, the service should indicate which execution modes are allowed.
-   If both execution modes are supported, the client should be able to specify the desired execution mode.
-   If the client does not specify an execution mode, the service should automatically choose an appropriate execution mode.
-   When a client specifies an unsupported execution mode, the service should return an error.

#### Data transfer by value and by reference

Clients may send and receive data in two different ways: (1) by reference and (2) by value. In brief, mixed modes are allowed. Typically, small atomic data such as integers, floating-point numbers, and short strings are submitted by value, while large inputs or outputs are usually provided by reference.

-   The service may accept input data by value and by reference.
-   When a provided data reference is not accessible, the service should return an exception.
-   The service should be able to return output data by value and by reference.
-   The supported output modes should be specified for each offered process.
-   If multiple output modes are supported, the user should be able to choose. If the user specifies an unsupported transfer mode, the service should return an exception.

#### Job monitoring

-   For asynchronous jobs, the service should provide machine-readable status information.
-   In general, the service should use the basic set of states in Table 3 to describe the current state of a process job.
-   The service should report the percentage of completion for a running job.
-   The service should provide an approximate time when the process results will be available.
-   The service should suggest an appropriate time for the next status query.
-   The service should report an expiration time for the job, after which the job identifier becomes invalid and existing resources will be removed from the server.