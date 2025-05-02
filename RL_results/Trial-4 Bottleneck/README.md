# Hybrid Control Strategy 

This phase addresses a critical exploration bottleneck observed during prolonged RL training. Despite effective positioning behavior, the agent fails to learn grasping. A hybrid control strategy is introduced to bridge this gap and ensure task completion.

---

## Results Summary

After extensive training (150,000+ timesteps) and reward tuning, the agent demonstrates consistent success in early stages but fails to achieve full task execution.

**Current Performance:**

* **Stage 1 (Positioning/Descent)**: Mastered with 100% consistency
* **Stage 2 (Grasping)**: Not reached
* **Test Success Rate**: 0.00%
* **Average Reward**: \~54.0 (up from 6.2 early in training)
* **Grasp Attempts**: 0 across all test episodes

These results suggest the agent has **reached a local optimum** by exploiting early-stage rewards without progressing toward grasping or lifting behaviors.

---

## Implementation Details

* **Task**: Robotic grasp-and-lift
* **Visual Input**: YOLO-based object detection
* **Action Space**: Continuous 6-DOF end-effector control + gripper state
* **Reward Structure**: Five-stage shaped reward function
* **Training Duration**: 150,000+ timesteps
* **Evaluation Metrics**: Lift height, average reward, grasp attempts

---

## Current Challenge: Exploration Bottleneck

The agent has encountered an **exploration bottleneck**, a known issue in RL where:

* The policy stabilizes around a locally optimal but incomplete behavior (e.g., hovering).
* The subsequent actions (grasping, descent) are **rarely explored** due to low probability and sparse rewards.
* **Increased training time does not lead to transition**, indicating a failure to generalize beyond early success stages.

Importantly, **perception and detection are not the problem**—the issue lies in the policy’s inability to explore further action combinations.

---

## Recommended Solution: Hybrid Control Strategy

To overcome this bottleneck while retaining learned positioning behavior, we adopt a **hybrid control approach**:

### Phase 1: RL-Based Positioning

* Use the trained RL policy to guide the arm to a **hover state above the object**.
* Covers **Stage 0 → Stage 1** using learned behaviors.

### Phase 2: Manual Grasp Execution

* Once the agent reaches a stable hover, trigger a scripted grasping sequence:

  * **Descend** by a fixed offset
  * **Close** the gripper
  * **Lift** the object by a fixed height
  * **Evaluate** grasp success via object displacement and gripper state

---

## Implementation Plan

To integrate hybrid control into the current environment:

1. **Stage Detection**

   * Enhance the environment to detect when the agent completes Stage 1 (hovering above object)

2. **Control Switch**

   * Initiate manual grasp logic when the RL policy stabilizes in the hover zone

3. **Manual Grasp Logic**

   * Execute:

     * Downward motion
     * Gripper closure
     * Upward lift
   * Record success based on:

     * Object height change
     * Gripper engagement
