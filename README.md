# Quantum Circuit Compilation Challenge: 8-Qubit QFT on a Penning Trapped Ion Architecture

Welcome to the challenge! Your goal is to compile a Quantum Fourier Transform (QFT) circuit for 8 qubits onto a Penning trapped ion quantum computer. You'll need to manage ion transport, gate scheduling, and noise considerations.

All the resources and materials for this challenge are available in the repository:  
[**ZuriQ Challenge: Quantum Hackathon 2025**](https://github.com/ZuriQ/quantum-hackaton-2025)

Feel free to clone the repository and submit issues if you encounter any. You can also ask all your physics, ion-trap, and quantum-related questions there or directly to the on-site ZuriQ representative.

## Updates and Additional Information

### General Updates
- **Pavel's Availability**: Pavel will be available at Fusion (where we had dinner on Friday) if you wish to find him for discussions or questions.
- **Repository Submission**: We recommend forking the repository and submitting your work via a pull request.
- **Initial Ion Positions**: Ions are free to have any initial positions as long as they respect the rules outlined in the challenge.
- **Fixed a typo in noise section**: We meant $p$ should be kept below 1 not $\bar{n}$

### MS Gate Definition
Our definition for the Mølmer–Sørensen (MS) gate is as follows:

$$ MS(\phi) = \exp\left(-i \frac{\phi}{2} (X \otimes X)\right) = \begin{bmatrix} 
\cos\left(\frac{\phi}{2}\right) & 0 & 0 & -i\sin\left(\frac{\phi}{2}\right) \\
0 & \cos\left(\frac{\phi}{2}\right) & -i\sin\left(\frac{\phi}{2}\right) & 0 \\
0 & -i\sin\left(\frac{\phi}{2}\right) & \cos\left(\frac{\phi}{2}\right) & 0 \\
-i\sin\left(\frac{\phi}{2}\right) & 0 & 0 & \cos\left(\frac{\phi}{2}\right) \end{bmatrix} $$

### Gate Decomposition Help
If you encounter difficulties decomposing the `RZ` gate into `RX` and `RY`, refer to the following resources:
- [Qiskit OneQubitEulerDecomposer](https://docs.quantum.ibm.com/api/qiskit/qiskit.synthesis.OneQubitEulerDecomposer)
- [PennyLane One-Qubit Decomposition](https://docs.pennylane.ai/en/stable/code/api/pennylane.ops.one_qubit_decomposition.html)

---

## What is a Penning Trap?

A Penning trap confines charged particles using a combination of a strong static magnetic field (for radial confinement) and a quadrupole electric potential (for axial confinement). The electric potential landscape can be fine tuned using microfabricated electrodes on a trap chip that is then embedded in a magnetic field that is homogenous over the whole trapping volume. The magnetic field is generated either using a superconducting magnet or very strong permanent magnets. 

Unlike the linear 1D arrays typically used in RF traps, Penning traps can naturally form 2D grids of trap sites that can be reconfigured (even in 3D!), limited only by the ability of the trap electrodes to shape the electric potentials. This reconfigurability is used to move ions in and out of interaction and memory zones where they are exposed or hidden from laser beams respectively in an approach know as the Quantum Charged Couple Device (QCCD) architecutre.  The added flexibility of the Penning trap can significantly reduce transport costs in the QCCCD approach and improve gate scheduling efficiency, as ions can take shorter paths to their destinations and avoid congestion at critical nodes. We at ZuriQ are just at the start of the journey and are eager to explore the possibilities unlocked by this architecture. This hackathon challenge will give you the opportunity to be creative and build compilation methods for a novel hardware architecture! 

To learn a bit more about how Penning traps work you can have a look at the references below. The underlying physics sets the limitations that we've modeled in this simplified example, but the knowledge of the classical equations of motion is not necessary for this challenge.

---

## References

### **Quantum Theory of the Penning Trap**
- [Quantum theory of the Penning trap](https://www.tandfonline.com/doi/full/10.1080/09500340.2017.1393570)

### **Papers from the ZuriQ Team**
- [Scalable Arrays of Micro-Penning Traps for Quantum Computing and Simulation](https://link.aps.org/doi/10.1103/PhysRevX.10.031027)  
- [Penning Micro-Trap for Quantum Computing](https://www.nature.com/articles/s41586-024-07111-x)  
- [A 3-Dimensional Scanning Trapped-Ion Probe](https://arxiv.org/abs/2412.17528)

### **Heating of Ions**
- [Ion-Trap Measurements of Electric-Field Noise Near Surfaces](https://link.aps.org/doi/10.1103/RevModPhys.87.1419)

### **QCCD Architecture**
- [Architecture for a Large-Scale Ion-Trap Quantum Computer](https://www.nature.com/articles/nature00784)  
- [Demonstration of the Trapped-Ion Quantum CCD Computer Architecture](https://www.nature.com/articles/s41586-021-03318-4)

---

## Your Task: Implementing the Compiler

Your task is to design and implement a compiler that translates the Quantum Fourier Transform (QFT) circuit for 8 qubits into a sequence of ion positions and gate operations that adhere to the trap architecture and physical constraints. The compiler needs to optimise and schedule single and two-qubit gates, while taking into account the 'cost' of reconfiguration, the limited coherence time of the qubits. and the trap geometry. Specifically, your compiler should:


1. **Decompose the QFT Circuit**:
    - Start with the standard QFT circuit for 8 qubits.
    - Decompose the QFT into its constituent gates (e.g. single-qubit rotations and controlled-phase gates).
    - Map these gates to the valid operations (`RX`, `RY`, `MS`) supported by the trap.

2. **Plan Ion Shuttling**:
    - Determine the initial positions of the 8 ions on the trap grid.
    - Plan the shuttling of ions to ensure that gates can be executed at the correct nodes while minimizing shuttling costs.
    - Ensure that all shuttling constraints (e.g., adjacency, no overlap) are satisfied.

3. **Schedule Gates**:
    - Assign time steps for each gate operation, ensuring that:
      - Single-qubit gates (`RX`, `RY`) are applied at `standard` nodes.
      - Two-qubit gates (`MS`) are applied at `interaction` nodes with the participating ions in place for the required duration.
    - Optimize the schedule to minimize the total time steps and temperature costs.

4. **Optimize for Fidelity**:
    - Minimize the temperature cost (see below for details) associated with ion shuttling to reduce noise applied on MS gates.
    - Ensure that the fidelity between the ideal QFT circuit and the compiled noisy circuit is as high as possible.

5. **Generate Outputs**:
    Your compiler should produce two main outputs: `positions_history` and `gates_schedule`. (see below for details)

    - `positions_history` tracks the location of each ion at every time step.
    - `gates_schedule` lists the quantum gates applied at each time step.

    To validate your outputs and ensure correctness, you can use the provided `verifier` and `fidelity` functions. Here's how you can integrate them into your workflow:

    ```python
    from verifier import verifier
    from fidelity import fidelity

    verifier(positions_history, gates_schedule, graph)
    fidelity(positions_history, gates_schedule, graph)
    ```

    - The `verifier` function checks if your `positions_history` and `gates_schedule` adhere to all the rules outlined in the challenge.
    - The `fidelity` function calculates the fidelity between the ideal (noise-free) quantum state and the noisy quantum state, helping you assess the performance of your compiled circuit.

    Make sure to run these functions after generating your outputs to ensure they meet the challenge requirements.

6. **Visualize the Compilation Process**:
    - Create a visualization (e.g., a video or diagram) that illustrates:
      - The shuttling of ions across the trap at each time step.
      - The execution of gates and their corresponding positions.

---

## Trap Architecture

1.  **Trap Geometry**:

    The trap used in this hackathon is a 5x7 grid, consisting of the following node types:

    - **Interaction Nodes**: 6 nodes where two-qubit MS gates can be applied.
    - **Standard Nodes**: 29 nodes where single-qubit RX/RY gates can be applied.
    - **Idle Nodes**: 29 nodes located above the standard nodes, where ions can idle at a lower cost.

    Refer to the implementation in `trap.py`. Instead of creating the graph from scratch, 
    use the `create_trap_graph` function provided in the file. This function generates the 5x7 grid structure of 
    the Penning trap, including the definitions of `interaction`, `standard`, and `idle` nodes.

    <p align="center">
        <img src="penning_trap_graph.png" alt="Penning Trap Graph" width="50%">
    </p>

---

## Compilation rules

1.  **Valid Gates**:
    *   `RX(angle, wire)`: Single-qubit rotation around X.
    *   `RY(angle, wire)`: Single-qubit rotation around Y.
    *   `MS(angle, wires=(w1, w2))`: [Two-qubit Mølmer–Sørensen gate](https://en.wikipedia.org/wiki/M%C3%B8lmer%E2%80%93S%C3%B8rensen_gate).
2.  **Gate Execution Rules**:
    *   **RX/RY Gates**:
        *   Can only be applied when an ion is at a `standard` node.
        *   Require **one unit of time** to execute.
        *   Cannot be applied at `interaction` or `idle` nodes.
    *   **MS Gates**:
        *   Must be applied when both participating ions are located at the *same* `interaction` node.
        *   Require **two units of time** to execute. During this period, the participating ions must remain stationary at the interaction node.
        *   Once the gate operation is complete, the ions must be moved away (immediately) from the interaction node.
    *   Gates can be executed in parallel, provided they do not involve the same ions.
3.  **Ion shuttling and Constraints**:
    *   Ions can only move between adjacent nodes in the graph (i.e., where an edge exists).
    *   A move between adjacent nodes takes **one unit of time**.
    *   **No Overlap**:
        *   Two ions cannot occupy the same `standard` or `idle` node at the same time.
        *   Two ions cannot occupy simultaneously the `standard` node and the `idle` above it.
        *   Two ions can share the same `interaction` node only when an MS gate is being applied to them at that specific time (two units of time). Additionally, no more than two ions are allowed at an `interaction` node simultaneously.
        *   During shuttling, if ion A moves from node X to node Y, no other ion can be at node Y at that time step. Exchange moves (A moves X->Y while B moves Y->X) are forbidden.

---

## Noise

A "temperature" cost is associated with each ion at each time step, reflecting the energy cost of its state/action:
*   **Staying at a `idle` node**: Cost increase of **0.01** for that ion.
*   **Being at a `standard` node** (either staying or moving to/from it, if not a `idle` node): Cost increase of **0.02** for that ion. This includes time steps where RX/RY gates are applied.
*   **shuttling**: If an ion moves from one node to another: Cost increase of **0.03** for that ion for that time step.
*   **MS Gate**: During an MS gate, the ions are at an `interaction` node. The cost for being at an `interaction` node is **0.02** per ion per time step.


The temperature of all ions is evaluated at each time step. This temperature directly impacts the performance of the MS gate. Specifically, the average temperature (in units of motional quanta) of the two ions involved in the MS gate, denoted as $\bar{n}$, is used to calculate the parameter $p$:

$$p = \frac{\pi^2 \eta^4}{4} \bar{n} (\bar{n} + 1)$$

Here:
- $\eta = 0.05$ is a fixed parameter.
- $\bar{n}$ represents the average temperature of the two ions involved in the MS gate.

While $p$ is theoretically unbounded, solutions where $p$ exceeds 1.0 are considered highly suboptimal and are unlikely to perform well in this challenge. The parameter $p$ is then used to apply a depolarizing channel to the two qubits involved in the MS gate operation, introducing noise proportional to the temperature.

The objective of this hackathon is to optimize the compilation process to ensure that the fidelity between the ideal (noise-free) quantum state and the noisy quantum state remains as close to 1.0 as possible.

---

## Deliverables

You must provide two main outputs:

1.  **`positions_history`**: A list of lists.
    *   The length of `positions_history` will be the total number of time steps your compiled circuit takes.
    *   `positions_history[t]` should be a list of length 8 (number of ions), where `positions_history[t][i]` is the node ID representing the position of the i-th ion at time step `t`.
    *   **Interaction Nodes**: Represented as `(row, col)`.
    *   **Standard Nodes**: Represented as `(row, col)`.
    *   **Idle Nodes**: Represented as `(row, col, "idle")`.

2.  **`gates_schedule`**: A list of lists.
    *   The length of `gates_schedule` must be the same as `positions_history`.
    *   `gates_schedule[t]` should be a list of gate operations occurring at time step `t`.
    *   Each gate operation should be represented as a tuple:
        *   **RX Gate**: Represented as `("RX", angle, wire_index)`, where `angle` is the rotation angle, and `wire_index` specifies the qubit.
        *   **RY Gate**: Represented as `("RY", angle, wire_index)`, where `angle` is the rotation angle, and `wire_index` specifies the qubit.
        *   **MS Gate**: Represented as `("MS", angle, (wire_index_0, wire_index_1))`. This should only be included at the time step when the gate operation begins, but make sure the ions stay in position for the right duration.
    *   If no gates are applied at a given time step `t`, the corresponding entry in `gates_schedule[t]` should be an empty list `[]`.
    *   Each gate should be listed only once at the time step it begins and should not be repeated for the duration of its operation.

    ### Example Output

    Here is an example of what your outputs might look like for a simple scenario:

    #### `positions_history`
    ```python
    positions_history = [
        [(0, 0), (0, 1), (1, 0), (1, 2), (2, 0), (2, 1), (3, 0), (3, 1)],  # Initial positions at t=0
        [(0, 0), (0, 1), (1, 1), (1, 1), (2, 0), (2, 1), (3, 0), (3, 1)],  # Qubit 2/3 moved to (1, 1)
        [(0, 0), (0, 1), (1, 1), (1, 1), (2, 0), (2, 1), (3, 0), (3, 1)],  # No movement at t=2
    ]
    ```

    #### `gates_schedule`
    ```python
    gates_schedule = [
        [("RX", 1.57, 0), ("RY", 3.14, 1)],  # Gates applied at t=0
        [("MS", 0.78, (2, 3))],              # MS gate applied at t=1
        [],                                  # (No gates applied at t=2) There is no need to repeat MS for its duration in the gates schedule.
    ]
    ```

    In this example:
    - At time step `t=0`, two single-qubit gates (`RX` and `RY`) are applied to qubits 0 and 1, respectively.
    - At time step `t=1`, a two-qubit `MS` gate is applied to qubits 2 and 3.
    - At time step `t=2`, `MS` is implicity applied, and the ions at (1, 1) need to remain stationary.

    This format ensures that your outputs are clear, valid, and adhere to the rules specified in the challenge.

3. Given `positions_history` and `gates_schedule` and the `graph`, create a visualization of the ion shuttling and gate interactions during the compilation process. This can be in the form of a video that illustrates:

* The movement of ions across the trap at each time step.
* The execution of gates (RX, RY, MS) and their corresponding positions.

This visualization will provide valuable insights into the behavior of your compiler and help refine your approach. It can also serve as a useful tool for communicating your solution to others.

---

## Evaluation Criteria

*   **Validity**: The `positions_history` and `gates_schedule` must adhere to all the rules outlined above (verified by the `verifier` function).
*   **Correctness**: The compiled sequence of gates must be equivalent to the original QFT circuit. (verified by the `verifier` function)
*   **Cost**: The quantum fidelity is one of the main criteria for evaluation, so calculate it using the `fidelity` function to assess how closely your compiled noisy circuit matches the ideal quantum state. Strive to maximize fidelity by minimizing temperature costs.
*   **Visualization**: The quality and clarity of the visualization video showcasing ion shuttling and gate interactions will also be considered. This video should effectively illustrate the compilation process and highlight areas for potential optimization.
*   **Presentation**: Clearly explain your compiler design, detailing each step of the process. Pay special attention to justifying your ion shuttling strategy.

---

## Getting Started

0. **Installation**:

    To get started, ensure you have Python 3 installed on your system. Then, install the required dependencies:

    ```bash
    pip install networkx pennylane==0.37.0
    ```
1.  Familiarize yourself with:
    *   The [8-qubit QFT](https://en.wikipedia.org/wiki/Quantum_Fourier_transform):
        - Understand the mathematical foundation and circuit representation of the Quantum Fourier Transform.
        - Review its decomposition into single-qubit rotations and controlled-phase gates.
    *   Review decomposition rule for one and two qubit gates into `RX`, `RY` and `MS`.
    *   The Penning trap architecture:
        - Review the rules and constraints for ion shuttling and gate execution as described in the "Trap Architecture" section.
        - Understand the cost model for ion temperature and its impact on noise.
    *   The trap graph implementation in [`trap.py`](./trap.py):
        - Explore the 5x7 grid structure of the Penning trap, including the definitions of `interaction`, `standard`, and `idle` nodes.
        - [NetworkX Documentation](https://networkx.org/documentation/stable/): Understand how to work with graphs and perform pathfinding.
    *   The `verifier` function in [`verifier.py`](./verifier.py):
        - Learn how the `verifier` checks the validity of `positions_history` and `gates_schedule`.
        - Review the constraints it enforces, such as adjacency, no overlap, and gate execution rules.
    *   The `fidelity` function in [`fidelity.py`](./fidelity.py):
        - Understand how the fidelity between the ideal and noisy circuits is calculated.
        - Review the impact of ion temperature and noise on the fidelity of the compiled circuit.

2.  Experiment with the provided code:
    *   Run the `verifier` function with sample `positions_history` and `gates_schedule` to understand its output.
    *   Use the `fidelity` function to evaluate the performance of a compiled circuit.

3.  Visualize the process:
    *   Create diagrams or animations to illustrate ion shuttling and gate interactions.
    *   Use visualization tools to debug and optimize your compiler.

Good luck!
