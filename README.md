# VPP-and-carben-credit
# 数据集
1.美国风能和太阳能发电量，从1983年开始的每年每月实时更新数据来源网站 This dataset, provided by the U.S. Energy Information Administration (EIA) in the Electric Power Monthly report, contains monthly data on wind energy production and other renewables in the United States. https://www.eia.gov/totalenergy/data/browser/?tbl=T10.01#/?f=A&start=1949&end=2023&charted=6-7-8-9-14 https://www.eia.gov/electricity/monthly/

2.美国平均电价，从1978年11月开始，每年每月实时更新数据来源网站 美国劳工统计局：美国各地区、各人口普查区域和选定大都市地区的平均能源价格 https://www.bls.gov/regions/midwest/data/averageenergyprices_selectedareas_table.htm

3.负荷（美国全部，居民，商业，工业，交通，其他）从2001年1月开始每年每月数据来源网站 This dataset, provided by the U.S. Energy Information Administration (EIA) https://www.eia.gov/electricity/data/browser/#/topic/5?geo=g&agg=0,1&endsec=vg

4.美国月平均气温，月平均降水量，从1895年到2025年，每年每月实时更新数据来源网站 https://climatereanalyzer.org/clim/noaa-us-monthly/?dm_id=us&var_id=t2&mon_id=13

5.碳排放因子， 2011到2024每年平均值，lb co2/Mwh https://www.epa.gov/climateleadership/ghg-emission-factors-hub

# Microgrid Optimization Model

This repository contains the implementation of a Microgrid Optimization Model designed to manage energy sources, battery storage, and grid interactions with the goal of minimizing costs while meeting customer energy demands.

## Model Overview

The model optimizes the energy flow in a microgrid system consisting of renewable energy sources (solar and wind), non-renewable energy sources, and battery storage. The primary objectives are to minimize the cost of purchasing non-renewable energy and to maximize revenue from selling energy while ensuring that customer demand is met.

## Model Assumptions

1. **Battery Efficiency**:
   - The battery retains its charge from one time state to the next, with no idle depletion.
   - The charging efficiency is \( 95\% \), meaning the actual charge rate is $0.95 \times \text{ChargeRate}$.
   - The discharging efficiency is \( 95\% \), meaning the actual discharge rate is $\frac{\text{DischargeRate}}{0.95}$.

2. **Energy Sources**:
   - Two source nodes: one supplying renewable energy (free) and the other supplying energy at the market price.

3. **Cost Minimization**:
   - The model prioritizes minimizing costs, which means the grid imports only enough energy to satisfy demand.

4. **Battery Constraints**:
   - The maximum charging and discharging rate is 250 kwh, considered before efficiency losses.
   - The battery has a minimum charge state of 200 kWh and a maximum charge state of 1900 kWh.

## Model Parameters

- **T**: Set of time indices from \( 0 \) to \( 281 \).
- **Max\_Cap**: Maximum capacity of the battery = \( 2000  kWh \).
- **Charge\_Eff**: Charging efficiency of the battery = \( 0.95 \).
- **Discharge\_Eff**: Discharging efficiency of the battery = \( 0.95 \).
- **Min\_C\_State**: Minimum charge the battery can hold = \( 200 kWh \).
- **Max\_C\_State**: Maximum charge the battery can hold = \( 1900  kWh \).
- **Max\_Charge\_Discharge\_Rate**: Maximum energy that can be charged or discharged = \( 250  kWh \).
- **Price**: The price matrix for energy bought or sold at each time index.
- **Demand**: Customer demand at each time index.
- **Solar\_Gen**: Solar energy generation at each time index.
- **Wind\_Gen**: Wind energy generation at each time index.
- **Initial\_Charge**: Initial charge of the battery = \( 500 kWh \).

## Model Decision Variables

- **B\_State**: Energy (kWh) stored by the battery at time index \( t \).
- **Charge\_R**: Energy (kWh) used to charge the battery at time index \( t \).
- **Discharge\_R**: Energy (kWh) discharged from the battery at time index \( t \).
- **Grid\_I**: Energy (kWh) imported into the grid at time index \( t \).
- **Grid\_E**: Energy (kWh) exported to customers to satisfy demand at time index \( t \).
- **Relative\_Flow**: Energy (kWh) imported but not stored in the battery, instead sent directly to grid export.
- **Charge\_Binary**: Binary variable restricting simultaneous charging and discharging of the battery.
- **Energy\_Bought**: Non-renewable energy (kWh) purchased at each time index \( t \).

## Objective Function

The objective is to minimize the total cost of purchasing energy while maximizing the revenue from selling energy that meets customer demand.

<!-- $$\text{Minimize} \quad \sum_{t \in T} \text{Price}_t \times \left(\text{Energy\_Bought}_t - \text{Grid\_E}_t\right)$$ -->
$$\text{Minimize} \quad \sum_{t \in T} \text{Price}_t \times (\text{EnergyBought}_t - \text{GridE}_t)$$

## Model Constraints

1. **Battery Constraint**:
   $$\text{BState}(t) = \text{BState}(t-1) + \text{ChargeEff} \times \text{ChargeR}(t) - \frac{\text{DischargeR}(t)}{\text{DischargeEff}} \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

2. **Minimum Battery State**:
   $$\text{BState}(t) \geq \text{MinCState} \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

3. **Maximum Battery State**:
   $$\text{BState}(t) \leq \text{MaxCState} \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

4. **Grid Import Constraint**:
   $$\text{GridI}(t) = \text{ChargeR}(t) + \text{RelativeFlow}(t) \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

5. **Grid Export Constraint**:
   $$\text{GridE}(t) = \text{DischargeR}(t) + \text{RelativeFlow}(t) \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

6. **Source Energy Constraint**:
   $$\text{GridI}(t) \leq \text{SolarGen}(t) + \text{WindGen}(t) + \text{EnergyBought}(t) \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

7. **Customer Demand Constraint**:
   $$\text{GridE}(t) = \text{Demand}(t) \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

8. **Charge Binary Constraint**:
   $$\frac{\text{ChargeR}(t)}{\text{MaxChargeDischargeRate}} \leq \text{ChargeBinary}(t) \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

9. **Discharge Binary Constraint**:
   $$\frac{\text{DischargeR}(t)}{\text{MaxChargeDischargeRate}} \leq 1 - \text{ChargeBinary}(t) \quad \text{for all } t \in \{0, 1, \dots, 281\}$$

## How to Use

1. Clone the repository:
    ```bash
    git clone https://github.com/buckyball99/Microgrid-Energy-Optimization.git
    cd Microgrid-Energy-Optimization
    ```

2. Install the necessary dependencies:
    ```bash
    pip install -r requirements.txt
    ```

3. Run the optimization model

4. Analyze the results:
    The results will be saved as output files, and relevant plots will be generated to visualize the energy flow, costs, and renewable energy utilization.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
