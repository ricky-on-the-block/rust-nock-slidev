# Results and Metrics

<v-clicks>

While the design space has not been exhausted, here are the current benchmarks using the largest documented input Nock Noun, <b>decrement</b>.

| <b>Implementation</b> | <b>Benchmarked Time</b> | <b>Relative Speedup to Slowest</b> |
|----------|----------|----------|
| Naive Box (clones everywhere) | ~1.42 ms | - |
| Optimized Box | ~350 μs | ~4x |
| Optimized Rc | ~50 μs | ~28x |
| Unsafe Raw Ptr | ??? | ??? |

</v-clicks>
