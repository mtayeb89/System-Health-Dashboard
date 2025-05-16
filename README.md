# System Health Dashboard

![Dashboard Preview](dashboard_preview.png)

## Overview

System Health Dashboard is a lightweight, real-time monitoring tool built with Python and Dash that provides a visual interface for tracking key system metrics. This application displays CPU usage, memory consumption, disk utilization, and network traffic in an easy to understand dashboard format.

## Features

- **Real-time Monitoring**: Continuously updates metrics at 2-second intervals
- **Multi-metric Tracking**: Monitors CPU, memory, disk, and network performance
- **Responsive Graphs**: Interactive time-series visualizations
- **System Information**: Displays basic system details including OS version and processor
- **Lightweight**: Minimal resource footprint
- **Cross-platform**: Works on Windows, macOS, and Linux

## Requirements

- Python 3.6 or higher
- Dependencies: 
  - dash
  - plotly
  - psutil

## Installation

1. Clone this repository:
```bash
git clone https://github.com/yourusername/system-health-dashboard.git
cd system-health-dashboard
```

2. Create and activate a virtual environment (optional but recommended):
```bash
# On Windows
python -m venv venv
venv\Scripts\activate

# On macOS/Linux
python -m venv venv
source venv/bin/activate
```

3. Install the required packages:
```bash
pip install -r requirements.txt
```

## Usage

To start the dashboard, simply run:

```bash
python main.py
```

Then, open your web browser and navigate to:
```
http://localhost:8050
```

For remote access, use the IP address of your machine instead of localhost.

## Configuration

You can modify the following parameters in the script:

- `max_points`: Maximum number of data points to display (default: 50)
- `interval`: Refresh interval in milliseconds (default: 2000)
- Port number (default: 8050)

## How It Works

The application uses a multi-threaded approach:

1. A background thread continuously collects system metrics using the `psutil` library
2. The main thread runs a Dash web server that displays the collected data
3. Interactive Plotly graphs update automatically to show the latest metrics
4. Thread-safe data structures ensure consistent updates

## Customization

### Changing Colors

You can customize the graph colors by modifying the `line=dict(color='#HEXCODE')` parameters in the `update_graphs` function.

### Adding New Metrics

To add additional metrics:

1. Create a new deque in the global variables section
2. Add code to collect the metric in the `collect_data` function
3. Add a new graph component in the app layout
4. Update the callback function to create and return the new graph

## Troubleshooting

### High CPU Usage

If the dashboard itself is consuming significant CPU resources, try:
- Increasing the refresh interval
- Reducing the maximum number of data points

### Connection Issues

If you cannot connect to the dashboard:
- Verify the correct port is being used
- Check if any firewall is blocking the connection
- Ensure the host parameter is set correctly (use '0.0.0.0' for all interfaces)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgements

- [Dash](https://dash.plotly.com/) - The web framework used
- [Plotly](https://plotly.com/) - Interactive visualization library
- [psutil](https://github.com/giampaolo/psutil) - System monitoring library

---

## Code Documentation

### Main Components

#### Data Collection

The `collect_data()` function runs in a separate thread and continuously gathers system metrics:

```python
def collect_data():
    global last_sent, last_recv, last_time
    
    while True:
        # Collect current time
        current_time = time.strftime('%H:%M:%S')
        
        # Collect CPU usage
        cpu_percent = psutil.cpu_percent(interval=1)
        
        # Collect memory usage
        memory = psutil.virtual_memory()
        memory_percent = memory.percent
        
        # Collect disk usage
        disk = psutil.disk_usage('/')
        disk_percent = disk.percent
        
        # Collect network usage
        net_io_counters = psutil.net_io_counters()
        current_time_sec = time.time()
        time_diff = current_time_sec - last_time
        
        bytes_sent = (net_io_counters.bytes_sent - last_sent) / time_diff / 1024
        bytes_recv = (net_io_counters.bytes_recv - last_recv) / time_diff / 1024
        
        # Update values with thread safety
        with data_lock:
            times.append(current_time)
            cpu_percentages.append(cpu_percent)
            memory_percentages.append(memory_percent)
            disk_percentages.append(disk_percent)
            network_sent.append(bytes_sent)
            network_recv.append(bytes_recv)
```

#### Dashboard Interface

The Dash application layout defines the UI structure:

```python
app.layout = html.Div([
    html.H1("System Health Dashboard"),
    
    html.Div([
        html.H3("System Information"),
        html.Table([
            html.Tr([html.Td("System:"), html.Td(f"{platform.system()} {platform.version()}")]),
            html.Tr([html.Td("Processor:"), html.Td(platform.processor())]),
            html.Tr([html.Td("Python Version:"), html.Td(platform.python_version())])
        ], style={'border': '1px solid #ddd', 'padding': '8px', 'width': '100%'})
    ], style={'marginBottom': 20}),
    
    # Graphs for CPU, memory, disk, and network...
    
    dcc.Interval(
        id='interval-component',
        interval=2*1000,
        n_intervals=0
    )
])
```

#### Graph Updates

The `update_graphs()` callback function updates all charts when triggered by the interval timer:

```python
@app.callback(
    [dash.dependencies.Output('cpu-graph', 'figure'),
     dash.dependencies.Output('memory-graph', 'figure'),
     dash.dependencies.Output('disk-graph', 'figure'),
     dash.dependencies.Output('network-graph', 'figure')],
    [dash.dependencies.Input('interval-component', 'n_intervals')]
)
def update_graphs(_):
    # Safely retrieve current data
    with data_lock:
        times_list = list(times)
        cpu_list = list(cpu_percentages)
        memory_list = list(memory_percentages)
        disk_list = list(disk_percentages)
        net_sent_list = list(network_sent)
        net_recv_list = list(network_recv)
    
    # Create and return the updated graph figures
    # ...
```

## Future Enhancements

- **Process Monitor**: Add detailed per-process resource usage
- **Alerts**: Implement threshold-based alerts for critical metrics
- **History**: Add ability to save and review historical data
- **Export**: Add functionality to export metrics as CSV or JSON
- **Custom Metrics**: Support for user-defined metrics and sensors
- **Authentication**: Add basic authentication for remote access
