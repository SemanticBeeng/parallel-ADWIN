
# Scalable Detection of Concept Drifts on Data Streams with Parallel Adaptive Windowing
This repository contains a description and source code for Scalable Detection of Concept Drifts on Data Streams with Parallel Adaptive Windowing (Parallel Adwin).

## Parallel Adwin at EDBT 2017
Parallel Adwin was first published at the [21th International Conference on Extending Database Technology (EDBT)](http://edbticdt2018.at/) in March 2018.  

**Abstract:**
Machine Learning (ML) techniques for data stream analysis suffer from concept drifts such as changed user preferences, varying weather conditions, or economic changes.
These concept drifts cause wrong predictions and lead to incorrect business decisions.
Concept drift detection methods such as adaptive windowing (Adwin) allow for adapting to concept drifts on the fly.
In this paper, we examine Adwin in detail and point out its throughput bottlenecks.
We then introduce several parallelization alternatives to address these bottlenecks.
Our optimizations increase the throughput of the original Adwin approach by two orders of magnitude.
Thus, we explore parallel adaptive windowing to provide scalable concept drift detection for high-velocity data streams with millions of tuples per second.

**Publication:**
- Paper: [Scalable Detection of Concept Drifts on Data Streams with Parallel Adaptive Windowing](https://github.com/TU-Berlin-DIMA/parallel-ADWIN/blob/master/paper/Scalable-Detection-of-Concept-Drifts-on-Data-Streams-with-Parallel-Adaptive-Windowing.pdf)

- BibTeX citation:
```
@inproceedings{grulich2018parallelADWIN,
  title={Scalable Detection of Concept Drifts on Data Streams with Parallel Adaptive Windowing},
  author={Grulich, Philipp Marian and Saitenmacher, René and Traub, Jonas and Breß, Sebastian and Rabl, Tilmann and Markl, Volker},
  booktitle={21th International Conference on Extending Database Technology (EDBT)},
  year={2018}
}
```

## How to run
Import the maven project into your favorite IDE, and run the MicroBenchmark.java as an application. You need to provide the following arguments in order to specify a variant of the Adwin algorithm and a type of data stream (see further below for an explanation of the arguments):

```adwinType changeType batchSize numConstant numChange adwinDelta ```

 This will perform a benchmark of the specified Adwin variant with the specified type of data stream. The two-step, JMH-based benchmark works as follows:
 * Initially, 20 warmup iterations are performed. In each iteration, a fixed number of tuples from the data stream are processed by Adwin. The warmup iterations trigger JIT compilation and improve accuracy and reproducability of the subsequent measurement iterations. **After the warmup iterations, the state of Adwin (and the data stream) are reset.** So, the measurement iterations start with an empty histogram.
 * 100 measurement iterations are performed. In each iteration, a fixed number of tuples from the data stream are processed by Adwin. **The state of Adwin (and the data stream) are maintained over all 100 measurement iterations.** Specifically this means that the size of the histogram maintained by Adwin may grow over multiple or even all iterations (until a concept drift is detected). This allows to measure the processing time for a fixed number of elements (=one iteration) relative to the size of the histogram at the start of the iteration.

The resulting output to the console shows the processing time for every iteration as well as the number of Adwin cut checks performed in every iteration.


### adwinType
This argument specifies the variant of Adwin to be benchmarked and may be any of `ORIGINAL SERIAL HALFCUT SNAPSHOT`.
* `ORIGINAL` is the original Adwin implementation by Albert Bifet.
* `SERIAL` is our improved reimplementation of the original Adwin algorithm.
* `HALFCUT` is the improved Adwin variant presented in our paper as "Half-Cut Adwin".
* `SNAPSHOT` is the improved Adwin variant presented in our paper as "Optimistic Adwin".

### changeType
This argument specifies the type of concept drift appearing in the data stream and may be any of `CONSTANT ABRUPT INCREMENTAL GRADUAL OUTLIER`.
* `CONSTANT` produces a data stream with constant tuple values (no concept drift).
* `ABRUPT` produces a data stream with abrupt concept drift.
* `INCREMENTAL` produces a data stream with incremental concept drift.
* `GRADUAL` produces a data stream with gradual concept drift.
* `OUTLIER` produces a data stream with constant tuple values, but regular outliers (no concept drift).

For further details about the different types of concept drift, we refer to [A Survey on Concept Drift Adaptation](http://www.win.tue.nl/~mpechen/publications/pubs/Gama_ACMCS_AdaptationCD_accepted.pdf), pp. 6 by J. Gama et al. All data streams are periodic (see the following arguments).

### batchSize
This argument specifies the number of tuples processed per iteration.

### numConstant
This argument specifies the number of constant tuple values that are generated by a data stream between concept drift (or outlier) events. Therefore, it allows to set the rate of concept drifts in the stream. It is ignored if changeType is `CONSTANT`.

### numChange
This argument specifies the number of tuples that an incremental or gradual concept drift lasts. Therefore it is only used if changeType is `INCREMENTAL` or `GRADUAL`.

### adwinDelta
This argument specifies the confidence value delta which is an input parameter for the Adwin algorithm and determines the sensitivity of Adwin to (perceived) concept drifts.