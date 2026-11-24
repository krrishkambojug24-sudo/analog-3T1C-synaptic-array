# Analog 3T1C Synapse Array with Leaky Integrate-and-Fire Neurons

This project simulates a 4×4 analog synapse array in LTspice. Each synapse uses three transistors and one capacitor (3T1C) to store and read an analog weight. A Python controller measures spike timing and applies charge or discharge pulses to selected weight capacitors.

**One neuron, N0, was trained in this experiment.** After ten presentations of the same input sequence, its first output spike occurred **4.946 µs earlier** than before training.

![N0 membrane voltage and comparator output before and after training](evidence/first-spike.png)

## Experiment

Four 5 mV input pulses were applied in the order `IN0 → IN1 → IN2 → IN3`. Each pulse lasted 20 µs, with 30 µs between successive rising edges. All 16 weight capacitors started at 1.622 V.

The synaptic read currents feed an inverting leaky integrator built around an ADA4807 op-amp. When the membrane voltage falls below the −1.5 V reference, the comparator produces an output spike. The controller timestamps the comparator’s rising edge at 1.65 V.

## How training works

1. Apply the input sequence and measure the four input rising edges and N0’s first output spike.
2. For each input, calculate `Δt = t_post − t_pre`. Within the learning window, an input arriving before N0 fires requests a weight increase. An input arriving afterward requests a decrease. A shorter interval produces a larger requested change.
3. Select the corresponding cell in N0’s row. Use LTspice to find a charge or discharge pulse width that produces the requested change in capacitor voltage.
4. Carry the updated weights into the next presentation. After ten presentations, disable programming and replay the original sequence to measure N0’s response.

## Measured results

| N0 measurement | Before training | After training |
|---|---:|---:|
| First output spike, measured from simulation start | 146.516 µs | 141.571 µs |
| Latency from the first input rising edge | 46.511 µs | 41.566 µs |

N0 fired during the **second input pulse** in both evaluations. IN0 and IN1 therefore preceded the output spike and their weights increased. IN2 and IN3 arrived after the spike and their weights decreased. On replay, N0 reached its firing threshold sooner.

| N0 synapse | Initial capacitor voltage | Final capacitor voltage |
|---|---:|---:|
| IN0 → N0 | 1.622 V | 1.643 V |
| IN1 → N0 | 1.622 V | 1.683 V |
| IN2 → N0 | 1.622 V | 1.564 V |
| IN3 → N0 | 1.622 V | 1.598 V |

## Explore the circuit and verify the result

| File | Contents |
|---|---|
| [4X4S.asc](schematic/4X4S.asc) | LTspice schematic of the 16-cell array |
| [array_template.cir](array_template.cir) | Circuit netlist used by the Python controller |
| [automate.py](automate.py) | Spike detection, STDP updates, write-pulse calibration, and simulation control |
| [config.json](config.json) | Input timing, voltage levels, learning parameters, and selected neuron |
| [Spike pairs](evidence/stdp_pairs.csv) | Measured pre- and postsynaptic times and requested weight changes |
| [Programming records](evidence/programming.csv) | Applied pulse widths and measured voltage changes |
| [Weight history](evidence/weight_history.csv) | Capacitor voltages across training presentations |
| [Before waveforms](evidence/before_waveforms.csv) · [After waveforms](evidence/after_waveforms.csv) | Simulation time, N0 membrane voltage, and comparator output |
| [Before output edges](evidence/before_output_edges.csv) · [After output edges](evidence/after_output_edges.csv) | Comparator timestamps used for the first-spike comparison |

## Run the simulation

Install **LTspice** and **Python 3.11** on Windows, then follow the [SPICE model setup instructions](models/README.md). From the repository root, run:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe automate.py --self-test
.\.venv\Scripts\python.exe automate.py --config config.json
```

The self-test checks the controller logic. The final command launches LTspice and saves the generated circuit decks, waveforms, spike times, and weight records in a timestamped folder under `results_stdp/`.
