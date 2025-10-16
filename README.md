# Quantum-Binary-Classification
Used Iris dataset to classify the data



### Explain the rationale behind the chosen preprocessing methods.
StandardScaler: features are zero‑mean/unit‑variance so AngleEmbedding rotates by comparable magnitudes across features. This avoids some features dominating the encoded angles and stabilizes optimizer step sizes.
Binary selection (setosa vs versicolor): simplifies the task to a 2‑class problem so the VQC needs only a single scalar readout (mapped to probability), reducing required circuit resources for this demo.
Practical note: when using angle embeddings you must scale/clip features into a sensible rotation range (e.g., ±π) or do PCA/feature selection if qubit count < original features. Standardization is a lightweight, generally good first step; PCA or selecting the most informative features can reduce circuit width and improve trainability.


### Evaluate the expressibility of each circuit to identify its representational limits.

Observed: Circuit A mean ≈ 0.0636, std ≈ 0.0589; Circuit B mean ≈ 0.0628, std ≈ 0.0604.
Interpretation: for 4 qubits the Hilbert space dimension D = 16, and the Haar average pairwise fidelity ≈ 1/D = 0.0625. Your means are essentially equal to 1/16, so both ansätze (at the chosen layer counts) sample state space close to Haar‑random — i.e., high expressibility.
Consequence: each circuit can, in principle, represent a very large class of quantum states (and therefore complex decision functions). However, high expressibility alone does not guarantee successful training or good generalization.

### Discuss how the architecture affects the ability of the circuit to represent complex decision boundaries.

Circuit A (AngleEmbedding + StronglyEntanglingLayers)
StronglyEntanglingLayers build deep, structured multi‑qubit entanglement per layer. This typically increases expressible entanglement and global correlations quickly.
Pros: compact entanglement, good at capturing global nonlinear features.
Cons: more prone to barren plateaus / small gradients as depth/parameter count grows; more fragile to noise on hardware.
Circuit B (AngleEmbedding + data re‑uploading + BasicEntanglerLayers)
Data re‑uploading injects classical features multiple times; effectively gives more nonlinear feature maps with shallower entangling blocks.
Pros: often improves learning capacity for small datasets, can reduce depth while increasing effective feature complexity, and can make training easier than a single very deep entangling block.
Cons: may have many parameters (depending on blocks) and still be sensitive to optimizer hyperparameters; entanglement per block is simpler than StronglyEntanglingLayers.

### Design limits

Single-qubit PauliZ readout collapses the circuit mapping to a one‑dimensional score. That can constrain the kinds of decision boundaries that are easy to represent (though the ansatz + embedding can still produce complex non‑linear mappings). Multi‑observable readouts or classical postprocessing of multiple expectation values can increase expressivity for classification.


### Performance comparison & strengths/weaknesses (expressibility, accuracy, practical factors)

Expressibility: both ≈ Haar → similar theoretical representational capacity at current layer settings.
Accuracy: if one circuit outperforms the other despite similar fidelities, the differences likely come from:
embedding strategy (re‑uploading vs single shot),
optimization landscape (fewer/clearer gradients, fewer barren plateaus),
effective number of trainable parameters and parameter coupling to the readout.
Practical considerations:
Scalability: StronglyEntanglingLayers grow in depth/complexity; harder to scale on noisy hardware. Data re‑uploading scales by repeating small blocks and may be more hardware‑friendly if each block is shallow.
Circuit depth & noise resilience: shallower circuits (or re‑uploaded shallow blocks) are more noise‑resilient. Highly expressive deep circuits are more sensitive to noise and may overfit small datasets.
Trainability: highly expressive, highly parameterized ansätze can cause barren plateaus and optimization difficulties on small datasets. Regularization, fewer layers, or better initialization / optimizers may help.

### Recommendations and next steps (practical experiments)

Compare gradient statistics (mean norm) during training to detect barren plateaus.
Run a layer/parameter sweep (vary n_layers_A, n_blocks for B) and track expressibility + validation accuracy to find a sweet spot.
Try PCA → embed into 2 PCs and visualize decision boundary in 2D (compare classical vs quantum maps).
Test alternative readouts (combine expectations of several Pauli operators or multiple wires) and small classical postprocessing layers.
If targeting hardware: favor fewer layers, local entanglers, and noise‑aware training (readout/mitigation).
Monitor generalization (train vs val accuracy) to detect overfitting from excessive expressibility.
