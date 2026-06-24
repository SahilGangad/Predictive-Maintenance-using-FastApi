# Predictive Maintenance Project Workflow

This document illustrates the end-to-end workflow of the AI4I Predictive Maintenance application, showing how data moves from the user interface through the backend to the machine learning model and back.

```mermaid
graph TD
    %% Entities
    User((User))
    Browser["Frontend Web UI\n(index.html)"]
    FastAPI["FastAPI Backend\n(main.py)"]
    Models[("ML Models & Artifacts\n(model/artifacts/)")]
    
    %% User Interaction
    User -->|1. Enters Machine Sensor Data| Browser
    
    %% Frontend to Backend
    Browser -->|2. POST /predict\nJSON: MachineInput Schema| FastAPI
    
    %% Backend Processing
    subgraph Backend Processing ["FastAPI Backend Processing"]
        FastAPI -->|"3. Parse & Validate Data\n(schemas.py)"| BuildFeatures["4. Feature Engineering\n(Calculates Deltas, Ratios)"]
        BuildFeatures -->|"5. Scale Features\n(scaler.pkl)"| Predict["6. Model Prediction\n(model.pkl)"]
        Predict -->|"7. Risk Assessment\n(Calculates Risk Level)"| FormatResponse["8. Format Response\n(PredictionResponse Schema)"]
    end
    
    %% Loading models
    Models -.->|Loaded on App Startup| FastAPI
    
    %% Response
    FormatResponse -->|9. Returns JSON Response| Browser
    Browser -->|10. Renders Risk Level & Message| User
    
    classDef ui fill:#e1f5fe,stroke:#0288d1,stroke-width:2px,color:#000;
    classDef api fill:#e8f5e9,stroke:#388e3c,stroke-width:2px,color:#000;
    classDef db fill:#fff3e0,stroke:#f57c00,stroke-width:2px,color:#000;
    classDef process fill:#f3e5f5,stroke:#7b1fa2,stroke-width:1px,color:#000;
    
    class Browser ui;
    class FastAPI api;
    class Models db;
    class BuildFeatures,Predict,FormatResponse process;
```

## Workflow Steps

1. **User Input:** The user fills out a form on the web interface (`index.html`) with machine parameters like air temperature, rotational speed, torque, etc.
2. **API Request:** The frontend sends a `POST` request containing a JSON payload to the backend's `/predict` endpoint.
3. **Validation:** FastAPI validates the incoming JSON against the `MachineInput` schema defined in `schemas.py`.
4. **Feature Engineering:** The backend constructs the final feature array by combining original inputs with engineered features (e.g., temperature delta, power in watts, speed-torque ratio) to match the model's expected format.
5. **Scaling:** The features are scaled using a pre-fitted scaler (`scaler.pkl`).
6. **Prediction:** The scaled features are passed to the trained Random Forest model (`model.pkl`) to predict failure and calculate failure probability.
7. **Risk Assessment:** The probability is mapped to a risk level (LOW, MEDIUM, HIGH, CRITICAL).
8. **Response Formatting:** The backend structures the response into a `PredictionResponse` schema.
9. **API Response:** The JSON response is sent back to the frontend.
10. **UI Update:** The frontend updates the user interface to display whether maintenance is required, the failure probability, and the assessed risk level.
