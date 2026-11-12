# Week 7 – Go-to-goal control

The **50** trajectory as a list of waypoints, with separate programs for the
TRIK Studio simulator and a real robot. The plots and comparison workflow are
in **[odometry_analysis.ipynb](odometry_analysis.ipynb)**.

## Files

| File | Description |
| --- | --- |
| `go_to_goal_sim.qrs` | Simulator program: draws 50 and logs odometry. |
| `go_to_goal_real.qrs` | Hardware adaptation using the source robot's geometry and wiring; requires calibration. |
| `odometry_gtg_sim.txt` | Actual TRIK 2026.3 headless simulator run of the included program. |
| `odometry_analysis.ipynb` | Planned vs simulated vs real path, corner errors, timing and discussion. Runs locally or in Colab. |
| `block_code_sim.txt` | Exact Expression-block code from the simulator project. |
| `block_code_real.txt` | Exact Expression-block code from the real project. |

`odometry_gtg_real.txt` is created by running the real program. It is deliberately
absent until real measurements exist. An optional repeat run can be saved as
`odometry_gtg_real_repeat.txt`; the notebook loads it when present.
No real-robot test or measured real-robot accuracy is claimed.

## Run the simulator

1. Open `go_to_goal_sim.qrs` in TRIK Studio and use the graphical diagram.
2. Reset the 2D model. Keep the saved initial heading (facing right).
3. Run. The program uses the corrected positive-y targets that draw 50 on screen.
4. Retrieve `odometry_gtg_sim.txt` from the runtime's working directory. Each run
   clears its previous log. Copy the new log beside the notebook to analyse it.

One unit is 10 cm. Change `unit` in **1. Settings and targets** to scale the route.
The 10-unit move right is split into 6 units with marker up and 4 with marker down.

## Simulator vs real robot

| Setting | Simulator | Real starting configuration |
| --- | --- | --- |
| Wheel radius `r` | 0.028 m | 0.041 m |
| Wheel spacing `W` | 0.154 m | 0.196 m |
| Left motor / encoder | M3 / E3 | M4 / E4 |
| Right motor / encoder | M4 / E4 | M3 / E3 |
| Arrival radius | 0.003 m | 0.015 m |
| Forward maximum | 25 | 25 |
| Forward command near target | Approaches 0 | Floor of 10 power points |
| Steering gain `Kt` | 35 | 20 |
| Allowed heading error for driving | 0.08 rad | 0.12 rad |
| Timer | 5 ms | 5 ms (actual loop takes longer) |
| Pen control | Virtual marker | No actuator commands |

Real values are starting settings, not validated tuning. The geometry and port
mapping match the friend's real project; measure/check your robot before use.
The lower steering gain, 1.5 cm arrival radius and forward floor are adaptations
for physical motion, not parameters copied from his controller.

## Run on hardware

1. Select the matching TRIK controller/connection in Studio. Confirm M4/E4 is
   left and M3/E3 right. Change the mapping if your wiring differs.
2. Measure wheel radius and wheel spacing and enter them in block 1. With wheels
   lifted, verify that positive commands mean forward and encoder counts increase.
3. Place the robot at the starting pose, with a clear area at least 1 m wide and
   0.8 m deep plus robot clearance. Start with the pen lifted/off the robot.
4. Transfer/run the graphical program using your normal TRIK workflow. Stop the
   run if it turns incorrectly or stalls. Check wiring/signs before tuning gains.
5. Copy `odometry_gtg_real.txt` from the controller's working directory to this
   folder. Preserve each run before restarting, since the program clears its log.
6. Run the notebook again. Do a second run and save it under the repeat filename
   above if you want a repeatability comparison.

The real program follows the trajectory; it does not automatically lift a physical
pen. Its marker positions are no-op Expression blocks named `Pen: manual / no
actuator`. They do not pause for manual action. To physically draw two separate
digits, replace these blocks with commands for your pen mechanism, or add explicit
pauses for manual pen handling. No servo port or pen mechanism was supplied.
The source hardware program also has no automatic physical pen-lift blocks.

The real program's motor floor can prevent a slow approach from stalling, but it
may need adjustment. Steering can still stall if its command is below the motor's
dead zone. Check that the robot can turn and reach a single target before a full run.

## Log and notebook

Rows are `x,y,time` with **cm, cm, ms**, no header. Both programs write the same
format. Time uses `time()-t0`, so it includes block execution and file-writing time.
Coordinates are encoder estimates, not an external measurement of true position.
The notebook displays y downward to match the working simulator drawing. Physical
orientation depends on the start pose and wiring; a camera comparison may need a
coordinate-frame alignment, but do not mirror measured data silently.

Locally: install `numpy matplotlib jupyter`, open the notebook and Run All.
In Google Colab: open the notebook, run the optional upload cell, and upload logs.
The notebook works with simulation data alone and adds real results when supplied.
Minimum distance to each planned corner is a descriptive metric, not proof of true
physical accuracy; encoder odometry cannot detect wheel slip by itself.



