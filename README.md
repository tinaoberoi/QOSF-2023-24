# QOSF-2023-24

## Title
Title: Evaluating Qiskit Aer on A100 GPU vs Qiskit Estimator on CPU: A Comparative Study Under Limited Compute Resources

## Abstract

This study presents a focused evaluation of quantum computing simulations using the Variational Quantum Eigensolver (VQE) algorithm for the Hydrogen molecule (H2) system, comparing the performance of Qiskit Aer on NVIDIA's A100 GPU, V100 GPU, T4 GPU, against Qiskit Estimator on a standard CPU, under limited computational resources. Emphasizing practical constraints faced in quantum research, we benchmark these platforms for the specific implementation of VQE, a pivotal algorithm in quantum chemistry. The analysis spans across computational efficiency, scalability, and adaptability to different hardware configurations. We further assess the resilience of these simulators in handling computational errors and disruptions, a critical aspect for reliable quantum simulations. The study also delves into the optimization capabilities within Qiskit, examining how different optimizers influence the VQE implementation on these contrasting platforms. Our findings provide essential insights for researchers and practitioners in quantum computing, guiding the optimal utilization of simulation tools for efficient and accurate quantum chemical computations, particularly in resource-limited scenarios.  

## Introduction
In order to understand a molecule properties such as ground state energy play an essential role, prediciting its chemical and physical properties.  Knowing the ground state energy helps determinte the stability and reactivity of a moleule, and how it interacts with another molecule. These properties can have help predict behavior of substances before synthesising in labs, thus saving time and resources in material science or drug discovery. 

However, solving for ground state energy of molecules is difficult on classical computers due to complex nature of interactions involved within a molecule. Molecular simulation problems grow exponentially with the increase in number of particles, making it infeasible for classical computers. This computational challenges can be solved using quantum computing algorithms like `Variational Quantum Algorithms`.

Variational Quantum EigenSolver (VQE) algorithms is a hybrod quantum-classical algorithm that uses variational technique to find minimum eigen values of a Hamiltonian. I have a VQE Tutorial explaining steps involved for VQE implementation from scratch, do check it out [here](https://github.com/tinaoberoi/Tutorial_VQE/blob/main/part2_tutorial.ipynb). This tutorial explains a step by step process for finding ground state energy of  `LiH` molecule.

In this project we implement VQE using qiskit both for H2 and LiH molecules.
## Approach for VQE qiskit implementation

1. Using Drivers to extracts information
   Qiskit has drivers that act as interfaces to classical chemistry. The available drives are as follows `PSI4Driver`, `PyQuanteDriver`, `PySCFDriver` are available.
   In our case we are using `PySCFDriver`. 
   
   ```
    geometry = (["H", "H"], [(0.0, 0.0, 0.0), (0.0, 0.0, 0.735)], charge=0, multiplicity=1)
    molecule = MoleculeInfo(geometry)
    driver = PySCFDriver.from_molecule(molecule, basis="sto3g")
    problem = driver.run()
    ```
2. Map molecular Hamiltonians to qubit Hamiltonians

    For these transformations here we have used `Jordan Wigner` transformations. Used as 
    ```
    # mapper
    mapper = JordanWignerMapper()
    qubit_op = mapper.map(problem.second_q_ops()[0])
    ```
3. Create an ansatz
    One of the crucial steps in VQE to final ground state is choosing the correct ansatz. Here we are using the `Unitary Coupled-Cluster Ansatz (UCC)`. 
    ```
      ansatz = UCCSD(
          problem.num_spatial_orbitals,
          problem.num_particles,
          mapper,
          initial_state=HartreeFock(
              problem.num_spatial_orbitals,
              problem.num_particles,
              mapper,
          ),
      )
    ```

4. Choosing Optimizers
    In accordance with the variational method, its parameters must be optimized to minimize the expectation value of the target Hamiltonian. We start with the optimizer popular for supressing noise `equential Least Squares Programming optimizer (SLSQP)`.

## Accelerating implementation using GPUs

How GPUs can help achieve better performance. Explain benchmarking results.

## Results
- Comparing Errors between Exact energy and GPU/CPU Energy

<img src="./images/T4_cpu_gpu_errors_comparison.png">


- Comparing Times between CPU and GPU times averaged over `50`

<img src="./images/T4_cpu_gpu_time_comparison.png">


As expected the GPU performed better than the CPU based simulators. The maximum performance achieved is `5x` using `SLSQP` optimizers.

For better understanding lets compare multiple optimizers for the best suitable optimizer for our use case.

<img src = "./images/optimizer_performance.png">

After compairing different optimizers, the top three performers both in terms of performance and errors are 

<ul>
  <li> ADAM
  <li> SLSQP
  <li> TNC
</ul>

For the next phase we compare how resilience level affect the performance while improving the error percentages on `TNC`.

- Compairing with resilience level
  
<img src="./images/T4_resilience_level_times.png">


<img src="./images/T4_resilience_level_errors.png">

## Summary of Data
  
  - The qiskit `cudastatevec` GPU estimator outperforms qiskit estimator in terms of error and time. 
  - The time and error comparison based on different resilience levels in qiskit is validated.
  - The best performing optimizer in terms of performance and error for VQE H2 use case is `SLSQP`

