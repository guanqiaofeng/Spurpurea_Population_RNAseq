# Find the best aligner

Based on paper "Simulation-based comprehensive benchmarking of RNA-seq aligners", to me, Novoalign, STAR and GSNAP are the best options. 

**Novoalign**
http://www.novocraft.com/documentation/novoalign-2/novoalign-user-guide/getting-started/

**STAR**
https://github.com/alexdobin/STAR

**GSNAP**
https://github.com/juliangehring/GMAP-GSNAP

1. Base-level precision and recall. These three program are among the best performers with different genome complexity.
2. Junction-level precision and recall. In low and middle genome complexity, STAR is the best one, GSNAP has better precision and Novoaglin has better recall. In high genome complexity, Novoaglin perform best, GSNAP and STAR has similar performance. As junction performance is important for splicing analysis, STAR would be the better option.
3. Effect of parameters. Novoalgin default parameters could yield the best performance already. GSNAP parameter tuning would slightly increase performance and STAR parameter tuning could increase maybe ~10% performance for the high complexity genome. 
4. Runtime. From short to long, STAR, GSNAP, Novoalign. Roughly, GSNAP is ~2 times of STAR AND Novoalign is ~8 times of STAR.

To summarize, **STAR** would be the best option for me, as it has good base-level and junction-level calling and short runtime. And I will need to tune parameters to achieve the best performance. 

## Getting started
