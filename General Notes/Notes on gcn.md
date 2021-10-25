# Notes on gcn/rdna/rdna 2 based on my understanding from slides and isa
## general architecture
- AMD graphics cards (based on talking to people at work gcn and rdna aren't all that different) Are made up of multiple Compute units (*CU*), each CU contains 4 simd processors with 16 threads each making for 64 threads running in total
- *important difference between gcn and rdna* on gcn these threads run in simd16 per clock, essentially round robing running the next simd processor on the next clock. RDNA runs properlly all 64 threads in lock step. 
- These groups of 64 threads are known as *waves*. Each CU can do about 10 waves based on *occupancy*.
- each thread allocates registers for scalar and vector values. Essentially values that will be the same across all threads will be *Scalar general purpose registers*(**SGPR**) and values that differentiate between threads will be *Vector general purpose register*(**VGPR**) 
- The idea is that each CU only has so many registers and each shader running on the CU must allocate from the same pool. How many registers being used is essentially occupancy **CLEAN UP THIS SECTION A BIT**
- The benefit of low occupancy is that in memory fetch situations (texture fetches, buffer fetching, etc) the CU will switch exectution to a differnt shader in order to continue execution without stalling on memory
- 