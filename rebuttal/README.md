# Supplemental Material for Rebuttal

## Time Distributions of Fuzzing Stages

See [this](./distributions).

## Profiling Verite

See [this](./flamegraphs).

## Coverage Improvement Case Study

We use DFS as a case study. DFS takes a fee on selling tokens but asks users to calculate the fee in advance, i.e. users can’t really sell 100 tokens even if users hold 100 tokens. Our mutation supports mutating the ratio of transferring and selling tokens and thus we can have higher code coverage by finishing the swapping successfully while ItyFuzz fails to cover the code [1]. 

- [1] https://bscscan.com/address/0x2B806e6D78D8111dd09C58943B9855910baDe005#code line 833-835
