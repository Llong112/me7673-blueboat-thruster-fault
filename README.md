# CORAL — BlueBoat Wall Inspection With a Weakened Thruster

## 1. What am I trying to find out?

**When one BlueBoat thruster loses effectiveness, how does the choice of controller affect inspection accuracy, recovery time, and required thrust?**

I’ll have the boat approach a straight virtual wall, stop at a chosen gap, and keep its bow facing the wall.

## 2. What does the simulator already give me?

BlueBoat’s two thrusters push the surface boat forward when their thrusts add and turn it when their thrusts differ. Neither points sideways, but the hull can drift sideways. The model includes inertia and drag, with provisional simulation values for both. [BlueBoat simulator README](https://github.com/Llong112/uvms-simulator_coral/blob/lance/blueboat/README.md) · [Planar dynamics model](https://github.com/Llong112/uvms-simulator_coral/blob/lance/blueboat/scripts/generate_blueboat_dynamics.py)

The planar pose $(x,y,\psi)$ gives position and heading. The body velocities are forward speed $u$, sideways speed $v$, and yaw rate $r$. I’ll use this simulated state and a known wall to design the controllers and test the fault.

## 3. What exactly are my two errors?

- **Distance error** is $e_d = d - d^*$. The gap $d$ is the shortest distance from the boat’s position to the wall, and $d^*$ is my target gap. Positive means too far away.
- **Heading error** is $e_\psi = \operatorname{wrap}(\psi - \psi_{\mathrm{face}})$. Wrapping gives the shortest signed angle from the desired direction to the bow. Counterclockwise is positive when viewed from above.

**How do I know $\psi_{\mathrm{face}}$?** I choose the wall and approach side. If the wall is at $x=0$ and I approach from $x<0$, I face it by pointing along $+x$, perpendicular to the wall, so $\psi_{\mathrm{face}}=0$. At any point along $x=-d^*$, facing $+x$ satisfies both goals.

## 4. Why do sine and cosine appear?

$$
\dot e_d = -u\cos(e_\psi) + v\sin(e_\psi), \qquad \dot e_\psi = r
$$

Picture the wall on the right. Point straight at it and all forward speed closes the gap, so $\dot e_d=-u$. Angle the bow slightly left and only the cosine component of $u$ closes the gap. Positive $v$ points 90° left of the bow, toward port. Its projection uses sine and opens the gap in this example. Since the wall is fixed, heading error changes at the yaw rate.

## 5. How do my lecture examples become a Lyapunov candidate?

“The boat can pass through the correct distance while it is still moving toward the wall.” As in the spring example, I need both displacement and speed.

$$
V_d = \tfrac12 k_d e_d^2 + \tfrac12 \dot e_d^2
$$

“The bow can pass through the correct heading while the boat is still turning.” As in the pendulum example, I need both angle and angular speed.

$$
V_\psi = k_\psi(1-\cos e_\psi) + \tfrac12 r^2
$$

The term $1-\cos e_\psi$ is zero when aligned, including after a full turn. Its slope is $\sin e_\psi$, which suggests a restoring turn in the opposite direction. The weight $w$ sets heading’s share of the candidate.

$$
V = V_d + w V_\psi, \qquad k_d,k_\psi,w > 0
$$

## 6. How would I test the controller and the fault?

With **input–output feedback linearization**, I’ll use the boat model to choose forward force and yaw torque so each error behaves like a damped system. A simpler **proportional–derivative (PD) controller** will request force and torque directly from the same errors and their rates.

I’ll first check the healthy case. Near the wall-facing heading and within motor limits, can I show that $V$ decreases during the approach and that both errors settle to zero?

I’ll inject the fault after the simulator splits the force and torque requests between the motors and limits each command. Port will deliver its limited thrust times $\alpha$. Starboard will deliver its full limited thrust.

**Fault-rule illustration.** If both motors are asked for 5 N and $\alpha=0.5$, port will deliver 2.5 N and starboard will deliver 5 N. Forward push will fall from 10 N to 7.5 N, and the imbalance will create an unintended turn.

## 7. What evidence will answer my question?

| Port effectiveness | Starts I’ll test with both controllers |
| --- | --- |
| 1.00 (healthy) | Near the target facing the wall, and farther away at an angle |
| 0.75 (partial loss) | Same two starts |
| 0.50 (partial loss) | Same two starts |

Both controllers will get the same starting pose and velocity, target, motor limits, and run duration. For each start, I’ll pick one fault time during powered approach and mark that instant in every run.

- **Path.** I’ll plot the top-down boat path with the wall and the target-gap line.
- **Accuracy and recovery.** I’ll plot both errors over time with the fault instant marked and compare the post-fault RMS error for each. I’ll time recovery from the fault until both errors stay within my proposed tolerances (5 cm and 5°) for the rest of the run. If a controller never recovers, I’ll report that explicitly.
- **Thrust demand.** I’ll plot commanded and delivered thrust for each motor and compare peak absolute commanded thrust between controllers.

## Current progress and next steps

I have the simulator, the error definitions, and a Lyapunov candidate. Next I’ll derive the controllers, check the healthy case, and run the paired fault tests.

## Sources and contributions

My notebook sketches supplied the wall geometry and fault idea. The spring and pendulum examples came from course material, and the boat dynamics came from the simulator linked in section 2. AI helped explain and organize the draft.
