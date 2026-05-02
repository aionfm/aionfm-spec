# Document 58 – UI Architecture & Design

## Purpose

This document outlines the architecture and design principles for the AionFM user interface.  The UI provides dashboards and tools for visualising forecasts, interacting with scenarios, exploring regimes and monitoring model health.  A well‑designed UI enhances usability and adoption.

## Audience

Front‑end developers, UI/UX designers and product owners for the `aionfm-ui` repository.

## Architectural Overview

The UI is a web‑based application built on modern front‑end frameworks (e.g. React, Vue) and communicates with the `aionfm-api` backend.  Key components include:

1. **Forecast Dashboard:** displays point forecasts, quantiles and intervals in line or area charts.  Supports selection of entities, variables, horizons and quantile levels.
2. **Scenario Explorer:** visualises multiple scenario paths.  Users can compare scenarios, highlight extreme trajectories and compute summary statistics (mean, percentile bands).
3. **Regime Viewer:** shows the current and projected regime states along the timeline.  Provides explanations for regime transitions and change‑point probabilities.
4. **Metadata Panel:** displays metadata about the selected entity, covariates and model configuration.
5. **Control Panel:** allows users to request new forecasts, adjust scenario counts, choose quantile levels and select covariates.  May include toggles for constraint enforcement and retrieval settings.
6. **Monitoring Dashboard:** for operations teams, includes panels for calibration metrics, error metrics, regime distribution and drift indicators.

## Design Principles

* **Clarity and readability:** use clean layouts and consistent typography.  Avoid clutter; present only relevant information by default with drill‑down options.
* **Interactivity:** allow users to hover or click on charts to see exact values, intervals and regime labels.  Provide filters and selectors to explore specific time periods or entities.
* **Accessibility:** ensure that colours and contrasts meet accessibility standards.  Provide keyboard navigation and screen reader support.
* **Responsiveness:** design for various screen sizes (desktop, tablet, mobile).  Use responsive charts that adapt to available space.
* **Modularity:** implement UI components as reusable modules.  Use a state management library (e.g. Redux, Vuex) to manage application state and API interactions.
* **Error handling:** display informative messages when forecasts cannot be generated (e.g. due to missing data).  Provide guidance on how to proceed.

## API Integration

* Use the API specification from Doc59 to interact with the backend.  Abstract API calls into a service layer with typed responses.
* Handle authentication and authorization if required.  Store tokens securely (e.g. HttpOnly cookies).
* Implement loading states and retry logic for API calls.  Use websockets or server‑sent events for real‑time updates where necessary (e.g. streaming scenarios).

## Visualisation Libraries

Choose well‑supported charting libraries (e.g. D3.js, Vega‑Lite, Plotly) that can handle large data volumes and interactive features.  Use components like line charts, area bands for intervals, scatter plots for anomalies and heat maps for regime distributions.

## User Roles

Define user roles with different access levels:

* **Analysts:** can view forecasts, scenarios and explanations; download data; provide feedback.
* **Planners:** can adjust covariate assumptions (e.g. promotions) and generate what‑if scenarios.
* **Operators:** can monitor calibration, set thresholds and trigger recalibration.
* **Administrators:** can manage users, update models and access logs.

## Implementation Considerations

* Use TypeScript for type safety and maintainability.
* Use component libraries (e.g. Material UI, Ant Design) for consistent UI elements.
* Implement testing with Jest and React Testing Library.  Use end‑to‑end testing (Cypress) for flows.
* Ensure that sensitive data is never exposed in the UI; fetch only necessary fields.
