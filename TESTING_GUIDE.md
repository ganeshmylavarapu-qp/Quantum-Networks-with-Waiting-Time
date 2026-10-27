# Waiting Time in Quantum Networks — testing guide

This guide explains how to run the companion app and check its calculations. The Markdown file is documentation; use the accompanying source ZIP to run the app locally.

## Run locally

1. Extract the source ZIP and open a terminal in the extracted folder.
2. Create an isolated Python environment (Python 3.10 or later):

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m streamlit run quantum_network_waiting_time.py
```

On Windows activate with `.venv\Scripts\activate` instead. Open the local URL printed by Streamlit. Package versions in requirements.txt record the environment used to prepare this download; other environments have not been validated.

## Assumptions

- Multiple requests are ready when scheduling begins. Their required elementary entangled pairs are already stored.
- Requests sharing a physical edge use distinct pairs; swapping consumes a pair. Sufficient memory slots are assumed.
- Each repeater performs at most one swap per interval. Shared repeater demand causes waiting.
- Generation retries, replenishment delays and random request arrivals are outside the scheduling calculation.
- Model 0: generation and swapping succeed deterministically; memory decay remains.
- Model 1: prepare links across the network before the requested paths are known. The success factor requires successful generation on every network edge.
- Model 2: prepare links for a known request along its selected path. Only those generation events enter its success factor.
- Successful swaps are ideal apart from their success probability. Failures contribute I_4/4 under the effective-state convention; this does not assert that physical loss produces a two-qubit state.
- The probability of obtaining one elementary pair is not the probability of stocking all pairs for all concurrent requests. Initial stocks are a scheduling assumption.

## Definitions and equations

Let n_P be the number of elementary links in a path, so it has n_P - 1 intermediate repeaters. All times must use consistent units.

- L: elementary-link length; alpha: loss in dB/km.
- t0 = L/v_fiber: full-link heralding storage time, neglecting processing delay. With refractive index 1.5, a 100 km link gives approximately 0.50 ms, and 20 km gives 0.10 ms.
- T: effective memory coherence time; delta_t: swapping interval.
- eta_d1: generation detector efficiency; eta_d2: swapping detector efficiency; eta_mem: memory efficiency; rho: source parameter; m: multiplexing attempts/modes.

```text
p_single = eta_d1^2 * rho^2 * 10^(-alpha*L/10)
p_elem = 1 - (1-p_single)^m
P_link (Model 1, uniform tree) = p_elem^(N-1)
P_link (Model 2) = p_elem^n_P
P_swap = (eta_mem*eta_d2)^[2(n_P-1)]
P_succ = P_link * P_swap
D = exp[-(n_P*t0 + tau_P)/T]
p_eff = P_succ * D
rho_eff = p_eff*|psi><psi| + (1-p_eff)*I_4/4
F_cond = (1+D)/2
F_FI = (1+p_eff)/2
```

For Model 0 set P_succ = 1. Attenuation uses full L: two photon half-link transmissions multiply to this factor. The baseline exposure is n_P*t0. Repeater-induced tau_P sums stored-pair intervals, including intermediate entangled pairs as swaps proceed. Maximum-waiting exposure uses n_P times the maximum repeater waiting count times delta_t.

## Check the calculation

1. Select five nodes and the chain topology. There are ten unordered source–target pairs: four direct links and six paths requiring swapping.
2. Open **Orders behind the maximum and minimum fidelity**. The six non-direct paths have 6! = 720 orders. Direct paths remain in the ten-path average but do not enter the swap ordering.
3. Inspect the displayed maximum and minimum orders, waiting tables, memory exposure, success factors and path fidelities. Sum all ten path fidelities and divide by ten. This must match the displayed average, within rounding.
4. Switch between repeater-induced and maximum waiting. Each prescription optimizes its own average over the same candidate orders; maximizing/minimizing orders may differ or tie. A same-order comparison is a separate diagnostic.
5. At delta_t = 0, ordering-dependent waiting vanishes. The two prescriptions must coincide for otherwise identical parameters. Nonzero t0 can still cause baseline decay.
6. At fixed settings, Model 2 must have fidelity at least as high as Model 1, since it requires a subset of the generation events. They coincide for a path using every network edge, or when p_elem = 1.
7. With P_succ = 1 and zero storage exposure, F_FI = 1. With p_eff = 0, F_FI = 1/2 under the effective-state convention.
8. Use this hand calculation: p_elem = 0.8, two path links, eta_mem = eta_d2 = 0.95, and D = 0.9. Model 2 gives P_link = 0.64; P_swap = 0.81450625; P_succ = 0.521284; p_eff = 0.4691556; F_FI = 0.7345778.
9. Download the path/order CSV tables and record all sidebar values when reporting a result. Graphs optimize at each swept interval; the detail table uses its own selected interval.

## Figures and limitations

Paper figure groups are shown by default. The **Show thesis plots** switch reveals preserved additional figures. The metropolitan map depicts one hub plus five peripheral nodes; the illustrative five-node numerical star is a different network size. Legacy critical-distance plots use a link-availability threshold of 0.99, not a teleportation-fidelity threshold, and are independent of waiting time.

For larger networks, order search may be sampled rather than exhaustive. Report the displayed search method and sample settings; sampled extrema are not guaranteed global extrema. This guide documents the current app and does not certify that every figure in an earlier PDF was regenerated with these conventions.
