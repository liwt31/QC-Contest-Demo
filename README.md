# Solution by Tencent Quantum Lab

The solution is in `get_energy.ipynb`.

Two circuits are used for this task, generated from `get_energy.ipynb` in qasm format. All three noise models share the same set of circuits. `circuit.txt` is the circuit for energy evaluation. `circuit_em.txt` is the circuit for error mitigation.

To install dependencies for the notebook, simply run `pip install -r requirements.txt`. Note `qiskit` must be the newest version.

Or directly run the notebook in binder:

[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/liwt31/QC-Contest-Demo/HEAD)


The key points of the solution are
- Ansatz Construction. Use hardware efficient $R_y$ ansatz to achieve extremely shallow circuit depth and short circuit duration
- Pauli Operator Grouping. Eliminate Pauli operators based on chemical heuristics.
- Error mitigation. Simple yet effective error mitigation with very little measurement overhead.

## Building the Parameterized Quantum Circuit
We use $R_y$ hardware efficient circuit with linear qubit connectivity and parallel CNOTs for minimal circuit depth.
The circuit consists of interleaved layers of $R_y$ and CNOT gates
$$|\Psi(\theta)\rangle=\prod_{l=k}^1\left [ L_{R_y}^{(l)}(\theta) L_{CNOT}^{(l)} \right ] L_{R_y}^{(0)}(\theta) |{\phi}\rangle$$
where $k$ is the total number of layers, and the layers are defined as
$$L_{CNOT}^{(l)}=\prod_{j=N/2-1}^1 CNOT_{2j, 2j+1} \prod_{j=N/2}^{1} CNOT_{2j-1, 2j}$$
$$L_{R_y}^{(l)}(\theta)=\prod_{j=N}^{1} RY_{j}(\theta_{lj})$$
Moreover, we append $X$ gates at qubit indices {2, 3, 4, 5, 8, 9, 10, 11} to ensure that the circuit outputs HF state when all parameters are set to zero.


## Group and Simplify the Pauli Operators
The ground state wavefunction of chemical systems is usually dominated by the HF state. This can be exploited to reduce the number of terms in the Hamitlonian. More specifically, we may combine terms such as $XX$ and $YY$ because they may have the same expectation value (up to a phase).
This simplification is not exact, but as shown in the results the accuracy is satisfactory.

Another benefit of the method is that it significantly reduces the number of terms that do not commute. In our solution, there're totally 20 measurement groups, in contrast to a total of 631 terms in the Hamiltonian.


## Error Mitigation
We use reference state error mitigation with HF state as the reference state. See https://pubs.acs.org/doi/10.1021/acs.jctc.2c00807 for details.
In short, we use quantum computers to estimate two energies $E(0)$ and $E(\theta)$. The first corresponds to estimated HF energy, and the second is for the estimated ground state energy.
The final energy is expressed as
$$E = E(\theta) + E_{HF} - E(0)$$
where $E_{HF}$ is the noiseless HF energy from a classical computer.

Thus, the method mitigates the error with one additional energy evaluation with the same circuit depth.


## Conclusion

The error rate of this solution is approximately 0.1%. The total number of shots is 240000. The circuit duration is 5024.

Since the total number of shots is below 1800000, we may increase the number of shots for each term. This is not considered here because slightly higher variance can do no harm :)

The key to the success is
- A shallow and efficient $R_y$ circuit to achieve short circuit duration and low error rate
- An efficient Pauli grouping method that greatly reduces the number terms as well as commuting groups
- Reference state error mitigation that has minimal overhead and offers high accuracy
