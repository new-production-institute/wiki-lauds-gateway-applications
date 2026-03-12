## Energy Analytics

The Energy Analytics module analyzes the relationship between machine operational parameters and electrical energy measurements. By correlating machine data with electrical parameters such as **active power (apower)**, **current**, and **voltage**, users can identify how different machine activities influence energy consumption.

The results help users understand machine energy behavior and identify parameters that have the strongest impact on power usage.

## Config Correlation

The **Config Correlation** feature allows users to configure and run correlation analysis between selected machine parameters and electrical measurements.

Users can:
- Select a **machine**
- Choose **machine parameters** (e.g., axis positions, fan status, temperatures)
- Select **electrical parameters** (**apower**, **current**, **voltage**)
- Define a **time range** for the analysis

After clicking **Analyze**, the system calculates correlations and displays them in a table.

Results are color-coded for easy interpretation:
- **Green** indicates a **positive correlation**
- **Red** indicates a **negative correlation**
- **Darker colors** represent **stronger correlations**, while lighter colors indicate weaker relationships.

Correlation values range from **-1 to +1**, where values closer to the extremes indicate stronger relationships between parameters.

You can visualise the corelation dashbord from the frontend that is running in Docker using a web server and is exposed on port 8088: http://localhost:8088/
![corealtion_dashboard](<media/corealtion_dashboard.jpeg>)