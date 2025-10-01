# AIDAO Round 1 task

This repository includes the baseline solution (both in Russian and in English) as well as the training dataset. The dataset is available int the "Releases" tab, see [this link](https://github.com/AIDAO2025/AIDAO2025-round-1/releases/tag/dataset).

## 1. Problem Description
Quantum cryptography is used for secure key exchange between participants, relying on fundamental laws of physics and proven mathematical reliability principles. In a secure data transmission system, a key stage is error correction, which compensates for signal distortions occurring during transmission through the quantum channel. Special low-density parity-check (LDPC) block codes are used for this purpose. They allow for correcting errors even under changing conditions: thermal fluctuations, equipment operation shifts, and physical impacts on the channel — all of which make the error rate time-varying. It is important not only to correct errors but also to flexibly adjust the parameters of the chosen code, even though the actual error rate prior to correction is unknown. 

Olympiad participants are tasked with developing an AI model that selects the most suitable LDPC code parameters for a demonstration quantum communication system. The code is defined by a triple of numbers $\{R, s, p\}$, where:  
$R$ — the ratio of useful bits to block length ($R \in \{0.5, 0.55, \dots, 0.9\}$),  
$s$ — the number of shortened nodes,  
$p$ — the number of punctured nodes.  

The model will receive as input the quantum channel characteristics and equipment state parameters. As output, it must suggest a parameter triple $\{R, s, p\}$ optimal for the current system state, ensuring efficient error correction in the generated key. The model's effectiveness can be evaluated using a provided simulator with data collected from real devices.  

**Goals:**  
- Minimize the number of BP-decoder iterations (increasing the throughput of the error correction procedure);  
- Reduce the amount of auxiliary information transmitted during correction;  
- Minimize the number of additional error correction rounds per frame (bit sequence).  

## 2. Technical Context
The transmitter generates quantum states and sends them through the quantum channel using randomly chosen polarization bases. At the sifting stage, participants (transmitter and receiver), exchanging information over a public classical channel, discard all states received in bases different from those used during transmission. As a result, a "sifted key" block (1,360,000 bits) is formed, which contains errors caused by channel noise, equipment features, or adversarial interference. This key is then post-processed, including error correction, verification, and privacy amplification, to form a shared secret key resistant to possible attacks.  

Error correction is one of the key post-processing stages, as both the length of the final key and the data processing speed depend on its efficiency. For correction, the system uses linear block LDPC codes. Each code is characterized by its rate $R$, calculated as the ratio of the size of the solution vector of the parity-check equations (syndrome) to the message length (32,000 bits). The available LDPC code pool includes rates $R$ from the set $\{0.5, 0.55, 0.6, \dots, 0.9\}$. Codes with lower rates correct errors more effectively but increase the volume of auxiliary information disclosed over the public channel, reducing the final key length.  

To improve flexibility and compensate for the limited set of codes, each data frame is supplemented with shortened ($s$) and punctured ($p$) nodes. The values of shortened nodes are known to both sides, while punctured ones are filled with noise. Adjusting $s$ and $p$ based on the current channel noise level when forming the frame makes it possible to optimize the use of the selected code with parameter $\alpha$ for efficient syndrome decoding. The sum $s + p$ is fixed at 4800, which defines the total frame size of 32,000 bits (27,200 bits are sifted key, the rest are additional nodes).  

## 3. Input Data
Participants are provided with time series of various parameter values characterizing the state of the quantum key distribution system in CSV format, obtained from real key generation on actual hardware:  
- Quantum Bit Error Rate (QBER) for decoy and vacuum states;  
- QBER for signal states from previous frames;  
- Number of sent/received states of each type (signal, decoy, vacuum);  
- Physical equipment parameters: voltages on the polarization controller, single-photon detector temperatures, intensity modulator voltages, etc.  

Each time-series record corresponds to values measured during accumulation of the corresponding sifted key frame. The most important factor for optimal code selection is the QBER estimate for signal states.  

The full CSV header is as follows:  
```python
block_id,frame_idx,E_mu_Z,E_mu_phys_est,E_mu_X,E_nu1_X,E_nu2_X,E_nu1_Z,E_nu2_Z,N_mu_X,M_mu_XX,M_mu_XZ,M_mu_X,N_mu_Z,M_mu_ZZ,M_mu_Z,N_nu1_X,M_nu1_XX,M_nu1_XZ,M_nu1_X,N_nu1_Z,M_nu1_ZZ,M_nu1_Z,N_nu2_X,M_nu2_XX,M_nu2_XZ,M_nu2_X,N_nu2_Z,M_nu2_ZZ,M_nu2_Z,nTot,unitsRatio,bayesImVoltage,opticalPower,polarizerVoltages[0],polarizerVoltages[1],polarizerVoltages[2],polarizerVoltages[3],temp_1,biasVoltage_1,temp_2,biasVoltage_2,synErr,N_EC_rounds,maintenance_flag,estimator_name,f_EC,E_mu_Z_est,R,s,p
```

## 4. Metric
The metric is the length of the secret key with a given probability of being unknown to an adversary. To obtain it, the errors in the sifted key must be corrected. If error correction fails, no secret key is produced — the soft real-time system fails to perform its function. In such a case, the solution will not meet the competition's time constraints. The dataset is divided into 5 parts for metric calculation:
- **Public metric** – secret key length (in bits) obtained using participants' selected correction codes on two fixed public data segments.  
- **Private metric** – sum of the public metric and secret key length (in bits) obtained on the three remaining data segments not used in the public metric calculation.  

## 5. Solution Requirements  
- **Code parameter adaptation:** For each frame, select the optimal $\{R, s, p\}$.  
- **Integration with simulator:** The algorithm must interact with the simulator available in the Contest. The simulator performs error correction on a frame using the chosen $\{R, s, p\}$ code on a fixed dataset and returns the efficiency metric: secret key length. The simulator is an isolated sandboxed component of the real sifted key post-processing procedure.  

## 6. Constraints  
The solution must operate in soft real-time mode.  

## 7. Materials  
Training data and a baseline solution are available at [this link](https://github.com/AIDAO2025/AIDAO2025-round-1).  
Due to technical limitations of GitHub, the dataset CSV can be found under "Releases" tab (see [this link](https://github.com/AIDAO2025/AIDAO2025-round-1/releases/tag/dataset))