Act as a Software Engineer and Data Analyst. Your task is to implement a "toy model" (simulation) in Python using synthetic data to optimize a Priority Queue (Max-Heap) system for VelozMart's logistics.

### 1. System Variables and Synthetic Data
Generate a synthetic dataset simulating the continuous arrival of orders. Each order must have:
- `priorityScore`: An integer in the range `[0, 100]`.
- `dispatchWindow`: An integer representing the remaining time for delivery, operating in the ranges `[-inf, 0)` and `(0, inf]`.
- `sizeCategory`: A size category that determines the `sizeBonus`. The mapping should be: category `P` receives a bonus of `n`, category `M` receives a bonus of `m`, and category `G` receives a bonus of `0`.

### 2. Scoring Function (Piecewise)
The Max-Heap must sort the orders by always extracting the maximum value. To handle the `dispatchWindow` mathematically without hitting zero, implement the following piecewise function to calculate the `dynamicScore`:
- For orders where `dispatchWindow > 0`: `dynamicScore = (W1 * priorityScore) + (W2 * (1 / dispatchWindow)) + sizeBonus`.
- For orders where `dispatchWindow < 0`: `dynamicScore = (W1 * priorityScore) + sizeBonus + (W3 * (-dispatchWindow))`.

### 3. Update and Re-queuing Mechanics
The simulation model must handle the passage of time by applying these rules:
- As time advances, decrease the `dispatchWindow`. If during the update `newDispatchWindow == 0`, you must mathematically force `newDispatchWindow = -1` to maintain continuity.
- Every 2-3 minutes (simulation time), the system must: recalculate the `dynamicScore` of all queued orders, insert incoming new orders, and execute a `heapify()` to restore the Max-Heap.
- **Concurrency Considerations (Optional but highly valued in your code design):** Simulate or account for the prevention of racing conditions. Ensure that there is always a heap available for read/extraction operations while the next one is being rebuilt and updated in the background.

### 4. Tracking Metrics
During the dispatch cycle, extract orders and track the following performance metrics:
- **Expired orders (Pedidos vencidos):** Record the exact time it expires. *(Note: The base design document literally states "When sizeBonus < 0 it becomes an expired order", but please treat this logically as a typo; it means when the `dispatchWindow` crosses the zero threshold into negative values).*
- **Delay time (Tiempo de atraso):** Calculate for expired orders using the formula: `(dispatch Time - expiredTime)`.
- **Throughput:** Record the dispatch time for each order to measure the processed volume.
- **Average dispatch time:** Calculate using the formula: `(dispatch Time - insertTime)`.

### 5. AI Objective
Build an optimization function (e.g., using Random Search or Grid Search) that evaluates the simulation iteratively. 
Find the best combination of weights (`W1`, `W2`, `W3`) and bonus values (`n` and `m`) that achieves the following:
1. Minimize expired orders and delay time.
2. Maximize throughput and prioritize the dispatch of high `priorityScore` orders.

Please return the complete Python script. Make sure to include comments explaining your approach to the optimization and finish with an execution block that prints the metrics of the best model.

## 6. Implementation Status

**What was built:** A Jupyter notebook (`model.ipynb`) implementing the full dispatch optimization pipeline across 10 cells.

**Architecture:** `Order` dataclass holds priority, window, and size category. `ScoringEngine` computes the piecewise `dynamicScore`. `DoubleBufferHeap` wraps two heaps with an atomic active/shadow swap for concurrency-safe rebuilds. A minute-by-minute simulation engine drives dispatch, tracks metrics (`compute_metrics`), and a grid search optimizer sweeps W₁ × W₂ × W₃ × n × m to find the best composite score.

**Key design decisions:** `dispatchWindow` is always positive at insert and forced to −1 at the zero crossing (per spec). Separate W₂ and W₃ govern each piecewise branch independently. Heaps rebuild every 2–3 minutes; dispatch capacity is configurable via the `CONFIG` dict.

**Current results** (seed 42): Best parameters are W₁ = 5.0, W₂ = 0.1, W₃ = 0.1, n = 20, m = 7. Against 800 orders over 480 minutes with 2 dispatches/min: 6 expired, 13.83 min average delay, 1.67 orders/min throughput, 48.23 average priority dispatched, composite score −103.11.

**How to use:** Tune `CONFIG` in Cell 1 (`N_ORDERS`, `SIMULATION_DURATION`, `DISPATCH_CAPACITY`, grid ranges, seed). Run all cells; the best model and metrics appear in Cell 8.

**Dependencies:** `numpy`, `pandas`, `matplotlib`, managed via pixi.