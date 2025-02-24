# Parallelizing the Mandelbrot Set

## Introduction
The Mandelbrot set comprises numbers \( c \) for which the iterative series \( z_{n+1} = z_n^2 + c \) remains limited when initiated from \( z_0 = 0 \). A complex number \( c \) is part of the Mandelbrot set if the series does not veer off towards infinity as \( n \) approaches infinity.

### Steps in Calculating the Mandelbrot Set

#### Grid Creation
First, a grid of numbers is established by defining a range for both the real and imaginary parts of \( c \).

#### Iterating
For each point \( c \) in the grid, a sequence is started with \( z_0 = 0 \). Then, it is iterated using the formula \( z_{n+1} = z_n^2 + c \) for a set number of iterations (e.g., 1000).

#### Bound Checking
After the iterations, if the magnitude of \( z_n \) exceeds a threshold (typically 2), it indicates divergence, and the point's color is determined based on how many iterations it took to diverge. Points that stay within bounds are colored black, indicating they are part of the Mandelbrot set.

#### Visualization
The assigned colors generate a representation of the Mandelbrot set, showcasing patterns that highlight its fractal nature.

The Mandelbrot set holds significance in various fields, including computer graphics, chaos theory, and mathematics. It illustrates how simple repetitive actions can lead to complex and sophisticated patterns.

Parallelizing the Mandelbrot set involves understanding different methods and addressing challenges such as load imbalance. Efficient parallelization is crucial for improving performance and reducing computation time.

## Implementation
To efficiently evaluate the Mandelbrot set through parallelization, Message Passing Interface (MPI) and OpenMP were used. The combination of the two leverages the benefits of their master-worker paradigms, maximizing performance.

### MPI
MPI computation follows the master-worker paradigm, where the master node manages the distribution of work across multiple worker nodes. The master node initializes parameters and distributes work in specific chunk sizes to the workers. Each worker node independently processes its assigned chunks by applying the Mandelbrot set formula \( z_{n+1} = z_n^2 + c \) iteratively.

### OpenMP
Within each worker node, OpenMP is used to further parallelize the process. By applying OpenMP, the workload for each worker node is divided among multiple threads.

This allows for effective utilization of multi-core processors:

```c
 double* times = (double*)malloc(threads * sizeof(double));
 #pragma omp parallel num_threads(threads)
 {
     double tstart, tend, telapsed;
     int tid = omp_get_thread_num();
     tstart = omp_get_wtime();

     #pragma omp for schedule(dynamic) nowait
     for (int i = start; i < end; i++) {
         for (int j = 0; j < nx; j++) {
             int index = i * nx + j;
             double x0 = xStart + (1.0 * j / nx) * (xEnd - xStart);
             double y0 = yStart + (1.0 * i / ny) * (yEnd - yStart);
             double x = 0, y = 0;
             int iter = 0;

             while (iter < maxIter) {
                 double temp = x * x - y * y + x0;
                 y = 2 * x * y + y0;
                 x = temp;
                 iter++;
                 if (x * x + y * y > 4) {
                     break;
                 }
             }
             matrix[index] = iter;
         }
     }
     tend = omp_get_wtime();
     telapsed = tend - tstart;
     times[tid] = telapsed;
 }
```

## Parallelization

### Grid Iteration
The outer loop iterates over rows indexed by \( i \) (representing the imaginary part), and the inner loop iterates over columns indexed by \( j \) (representing the real part). Each point in the complex plane computes the corresponding \( c \) value from \( x_0 \) and \( y_0 \), derived from \( xStart \), \( xEnd \), \( yStart \), and \( yEnd \).

### Mandelbrot Iteration
- Initialization of \( x \) and \( y \) to 0 corresponds to iteration with \( z_0 = 0 \).
- The while loop implements the iteration \( z_{n+1} = z_n^2 + c \).
- The condition \( x^2 + y^2 > 4 \) checks for divergence, as exceeding 2 (or 4 in squared terms) indicates divergence.

### Storing Results
The `matrix[index] = iter;` statement stores the number of iterations before divergence or reaching the maximum iteration count, used for coloring the Mandelbrot set visualization.

### Timing
Execution time for each thread is measured using `omp_get_wtime()`, providing insights on workload scheduling. The `schedule(dynamic)` directive dynamically distributes the workload among threads, improving load balancing and resource usage, leading to reduced wait time and faster execution.

## Load Imbalance and Chunk Size Optimization

### Load Imbalance
Load imbalance arises when workload is evenly distributed among workers but leads to inefficiencies due to variations in computation complexity.

### Testing Chunk Sizes
To address this, different chunk sizes were tested, and execution times recorded. The lowest execution times occurred at chunk sizes of 40 and 400. Selecting 200 as an optimal chunk size improved performance.

### Load Balance
With the optimal chunk size determined, execution time was assessed to achieve better load balance.

## Benefits of Parallelization
Increasing the number of workers significantly reduces execution time, demonstrating the efficiency of MPI and parallelization. The master-worker paradigm enables simultaneous task processing, maximizing resource utilization and minimizing wait time. This scalability ensures continued performance improvement as more workers are added.

## Visualization
After parallelization, the Mandelbrot set can be visualized, highlighting its fractal structure and intricate patterns.

## Conclusion
Parallelizing the computation of the Mandelbrot set improves performance and efficiency. The integration of MPI and OpenMP enables effective workload distribution, reducing execution time and maximizing resource utilization.

The process includes grid creation, iteration, and divergence checking. Addressing load imbalance is essential, as varying work complexity can lead to inefficiencies. Through careful analysis of chunk sizes and dynamic workload distribution, a balanced workload is achieved.

MPI and OpenMP significantly enhance performance, with optimized chunk sizes improving load balancing and execution time. These findings highlight the effectiveness of parallel computing in handling complex mathematical computations.
