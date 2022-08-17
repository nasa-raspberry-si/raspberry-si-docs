# Demo Scenarios

This document provides the test case specification for the JPL demo. Please refer to the [test and evaluation protocol](./evaluation-protocol.md) for more explanation about the test stages and the template for test case specification.

* Mission Description: Reach a location to excavate and scoop one sample from that location.

* Test Stages:

1. **Baseline A** 

	**Description:** No fault, no Autonomy

	**Mission Specification**:
		ARM_UNSTOW
		ARM_MOVE_CARTESIAN
		ARM_MOVE_CARTESIAN_GUARDED
		ARM_MOVE_CARTESIAN_GUARDED (retracting=True)
		TASK_SCOOP

	**Expected Behavior:** The arm should successfully reach the location defined in the mission specification.


2. **Baseline B**
	
	**Description:** Fault, no Autonomy

	**Mission Specification:**
		ARM_UNSTOW
		ARM_MOVE_CARTESIAN
		ARM_MOVE_CARTESIAN_GUARDED
		ARM_MOVE_CARTESIAN_GUARDED (retracting=True)
		TASK_SCOOP

	**Fault:** A fault (ARM_GOAL_ERROR) is injected randomly while performing the mission through `axclient`.

	**Expected Behavior:** Mission should not succeed. In other words, the mission should not be able to maintain the intent. 
	Observed Behavior: The arm could not continue with the rest of the actions when a fault was received by the testbed.	

3. **Challenge Stage C.A** 

	**Description:** Fault, Autonomy, Planner Knows All Fixes for possible Faults
	
	**Mission Specification:**
		ARM_UNSTOW
		ARM_MOVE_CARTESIAN
		ARM_MOVE_CARTESIAN_GUARDED
		ARM_MOVE_CARTESIAN_GUARDED (retracting=True)\
		TASK_SCOOP
	
	**Fault:** A fault (ARM_GOAL_ERROR) is injected randomly while performing the mission through `axclient`.

	**Expected Behavior:** Mission should succeed. After a fault is injected the autonomy should detect the fault and use the design-time specified fixes (cleaning the injected fault) and finish the mission.	
	
4. **Challenge Stage C.B1** 
	
	**Description:** Fault, Autonomy, Planner Does not the fixes; Planner synthesize a new plan at runtime to fix the fault.
	
	**Mission Specification:**
		ARM_UNSTOW
		ARM_MOVE_CARTESIAN
		ARM_MOVE_CARTESIAN_GUARDED
		ARM_MOVE_CARTESIAN_GUARDED (retracting=True)
		TASK_SCOOP

	**Fault:** A fault (ARM_GOAL_ERROR) is injected randomly while performing the mission through `axclient`.

	**Expected Behavior:** Mission should succeed. After a fault is injected, the autonomy should detect the fault and stop the arm, then it queries the PRISM planner and the new plan is generated at runtime, and finally executes the new plan.

 5. **Challenge Stage C.B2** 

 	**Description:** Behavioral Fault, Autonomy, Planner Does not the fixes; Planner synthesize a new plan at runtime to fix the fault
	
	**Mission Specification:**
		ARM_UNSTOW
		ARM_MOVE_CARTESIAN (intentionally setting a wrong z coordinate value that will trigger a fault, surface not reachable)
		ARM_MOVE_CARTESIAN_GUARDED
		ARM_MOVE_CARTESIAN_GUARDED (retracting=True)
		TASK_SCOOP

	**Fault:** We inject a different fault class, which we call a behavioral fault. Intentionally injecting a behavioral fault, by setting a wrong `z` coordinate value as the goal for the `ARM_MOVE_CARTESIAN`. The action that will trigger a fault, is because the surface is not reachable by the ARM for scooping and this produce a behavioral fault, e.g., due to measurement noise, the goal target was determined incorrectly.
	
	**Expected Behavior:** Mission should be a success. After a fault is injected the autonomy should detect the fault and stop the arm, then it queries the PRISM 	planner and the new plan is generated at runtime and finally executes the new plan. 
	till it finds the correct "z" value or "force_threshold"

	**New Plan (generated at runtime):** the new plan contains the correction of input by adjusting the z values in the coordinate that will be sent to the arm for finding the right `Pose` and `force_threshold` for sample collection. In particular, `ARM_MOVE_CARTESIAN` will be called with the correct inputs for the `z`  value in `pose` coordinate and `ARM_MOVE_CARTESIAN_GUARDED` with a correct input `force_threshold`.