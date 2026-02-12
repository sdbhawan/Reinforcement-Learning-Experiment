# Smooth & Safe Aircraft Pitch Control: From Pure RL to Hybrid LQI-TD3

This repository explores Deep Reinforcement Learning (DRL) for aircraft longitudinal control. The project focuses on tracking a pitch rate ($q$) reference signal using two distinct approaches: a pure incremental DRL agent and a hybrid supervisor-based controller.

##  Repository Structure

* **`TD3_incremental_working.ipynb`**: Implementation of a Twin Delayed DDPG (TD3) agent using **Incremental Control** and **CAPS Regularization** to achieve smooth actuator movements.
* **`LQI_TD3_incremental_working.ipynb`**: An advanced **Hybrid Controller** that uses a Linear Quadratic Integral (LQI) supervisor to provide a safety baseline, with the TD3 agent acting as a residual corrector.

---

##  The Core Challenge: Short-Period Dynamics
In both experiments, the environment simulates the **Short-Period Aircraft Dynamics**.
- **The Task:** Control the elevator deflection ($\delta_e$) to make the pitch rate ($q$) follow a sinusoidal or square-wave reference.
- **Incremental Control:** Instead of outputting an absolute position, the agent outputs a **$\Delta \delta_e$ (change in position)**. This ensures the control surface moves smoothly and adheres to physical rate limits.

---

## Phase 1: Incremental TD3 + CAPS
The first phase focuses on making RL "flight-ready" by addressing the "chatter" problem (high-frequency oscillations common in RL).

### Key Features:
* **TD3 Algorithm:** Uses twin critics to prevent overestimation of value functions.
* **CAPS (Cooperative Action Smoothing):** Adds two loss terms to the Actor:
    1.  **Temporal Smoothness:** Penalizes large changes between consecutive actions.
    2.  **Spatial Consistency:** Ensures similar flight states result in similar control outputs.
* **Result:** A controller that is much "quieter" on the actuators than standard RL.

---

## Phase 2: Hybrid LQI + TD3 (Residual RL)
The second phase introduces a **Safety Shield** using classical control theory. This is the "LQI" version.

### What does the LQI Supervisor add?
1.  **Integral Action:** By adding an **Integral Error State** ($\int q_{err} dt$), the controller mathematically guarantees zero steady-state error.
2.  **Residual Control:** The LQI controller calculates a stable baseline ($u_{LQI}$). The RL agent then calculates a small correction ($\delta_{RL}$).
3.  **The "Safety Shield":**
    - The final command is: $u_{safe} = u_{LQI} + \text{clip}(u_{RL} - u_{LQI}, -0.02, 0.02)$
    - This limits the RL agent's error; it can only "tweak" the LQI baseline by $\pm 0.02$. This prevents the agent from performing unsafe maneuvers while allowing it to optimize performance beyond what linear control can do.

---

## Comparison of Approaches

| Feature | Phase 1 (Pure RL) | Phase 2 (Hybrid LQI-RL) |
| :--- | :--- | :--- |
| **Stability** | Learned from scratch | Guaranteed by LQI baseline |
| **Steady-State Error** | Low (learned) | **Zero** (via Integral action) |
| **Actuator Wear** | Low (via CAPS) | Very Low (LQI + Clipping) |
| **Safety** | Experimental | **High** (RL is constrained) |

---

## Getting Started

### Prerequisites
```bash
pip install torch numpy gymnasium scipy matplotlib cvxpy
