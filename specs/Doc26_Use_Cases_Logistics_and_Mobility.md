## Document 26 ‒ Use Cases – Logistics & Mobility

### Purpose

To explain how AionFM can be applied to logistics and mobility forecasting, including shipment volumes, delivery times, route congestion, fleet utilisation, warehouse throughput and passenger demand.  Transportation systems exhibit complex spatio‑temporal dynamics, and robust forecasting supports route planning, capacity management and disruption handling.

### Intended Audience

Logistics planners, fleet managers, transportation analysts and mobility service providers deploying AionFM.

### Key Sections

* **System characteristics** – network structures, congestion, stochastic travel times.
* **Data sources** – shipment data, GPS traces, traffic sensors, public transit ridership.
* **Covariates** – weather, events, infrastructure status, pricing, fuel costs.
* **Disruption planning** – planning for outages, strikes, severe weather and accidents.
* **Scenario usage** – capacity planning, route optimisation and resilience analysis.

### System Characteristics

Logistics and mobility systems involve flows through networks of hubs, depots and routes.  Key characteristics include:

* **Network topology** – shipments and passengers travel through hubs and links; congestion and delays depend on network capacity and demand.
* **Stochastic travel times** – influenced by congestion, accidents, weather and road works.
* **Seasonality and events** – demand peaks during holidays, sale events or festivals.
* **Heterogeneous entities** – different types of shipments (perishable, hazardous), vehicle classes, passenger segments.

### Data Sources

* **Shipment data** – booking records, pickup and delivery timestamps, route information, weights and volumes.
* **GPS traces and telematics** – vehicle locations, speeds and routes.
* **Traffic sensors** – real‑time congestion levels, road closures.
* **Public transit data** – ridership counts, schedule adherence.
* **External datasets** – weather forecasts, event calendars, fuel price indices.

### Covariates

Include:

* **Weather** – heavy rain or snow slows travel and reduces demand.
* **Events** – sports games, concerts, trade fairs causing spikes in passenger demand or road closures.
* **Infrastructure status** – road construction, bridge closures, lane restrictions.
* **Fuel prices** – influence demand for ride‑sharing versus public transit.
* **Economic indicators** – industrial production affecting freight volumes.

### Disruption Planning

Logistics networks are vulnerable to disruptions:

* **Severe weather** – storms, hurricanes, floods.
* **Strikes and labour disputes** – port closures, trucker strikes.
* **Accidents and incidents** – crashes blocking major routes.
* **Equipment failures** – vehicle breakdowns, warehouse system failures.

AionFM’s scenario generation allows planners to simulate these disruptions by forcing certain regime tokens (e.g. “shock and recovery”) at specific times and assessing the impact on delivery times, fleet utilisation and warehouse throughput.  Planners can develop contingency plans, rerouting strategies and capacity buffers.

### Scenario Usage

Use AionFM scenarios to:

* **Plan capacity** – estimate fleet size and scheduling under various demand scenarios.
* **Optimise routes** – evaluate alternative routes given congestion forecasts and fuel prices.
* **Assess resilience** – quantify the impact of disruptions on service levels and costs.
* **Coordinate multimodal flows** – forecast passenger demand across ride‑sharing, public transit and micro‑mobility; plan balanced supply across modes.

AionFM’s ability to model cross‑series dependencies, integrate covariates and generate scenarios makes it a versatile tool for logistics and mobility.