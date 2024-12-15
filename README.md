
# Dynamic Scene Items Manager: A Transform Dock Plugin

This project is a free web-based tool designed to function as an **OBS Dock Page**, allowing users to dynamically transform and crop scene items directly within the OBS interface. It connects to OBS Studio via WebSocket, fetches existing values for scene items, and applies changes in real-time as users modify input fields. The dock page provides a web interface for selecting scenes and associated data sources using the OBS WebSocket protocol. It is particularly useful for adjusting data sources overlaid on other data sources, positioning them alongside other data sources, or correcting their transform values for enhanced visual presentation. Additionally, preset values allow for dynamically transforming data sources based on saved position, scale, rotation, and crop settings.

The intuitive interface allows for precise control of properties like position, scale, rotation, and cropping, making it ideal for streamers and content creators who require seamless and efficient scene management.

The tool includes 4 set Save/Retrieve Buttons that enable saving and retrieving scene item transformations using localStorage for quick access and reapplication. Each button corresponds to a specific localStorage key, ensuring efficient management of multiple transformations.

---

## Features

- **OBS Dock Integration**: Use this tool directly within OBS as a custom dock.
- **Real-Time Transformations**: Dynamically update position, scale, rotation, and crop values for scene items.
- **Scene/Data source selection**: Dynamically fetch and display a selectable list of scenes and their data sources.
- **Input Auto-Fill**: Automatically fetch and display existing values for the selected scene and data source.
- **Instant Updates**: Changes are applied immediately upon modifying input fields, with no need for refresh or extra clicks.
- **Efficient Management**: Manage up to 4 transformation presets for dynamic scene control.

---

## Technologies Used

- **HTML**: For the user interface.
- **JavaScript**: For connecting to OBS WebSocket and handling real-time transformations.
- **OBS WebSocket API**: To communicate with OBS Studio.
- **Local web server**

---

## Setup Instructions

### Prerequisites

- **OBS Studio**: Ensure version **28.0+** OBS Studio is installed.
- **OBS WebSocket**: OBS Studio version **28.0+** includes **the WebSocket server v5 or later** by default. Ensure it is enabled in OBS settings.

### Steps

- Clone this repository
- Navigate to the project directory
- Host the project using a local server
- Add the hosted page as an OBS Dock

---

## Configuration

### OBS WebSocket Settings

- Open OBS Studio.
- Navigate to **Settings > WebSocket Server Settings**.
- Ensure the WebSocket server is enabled and note the WebSocket URL and port (default: `ws://localhost:4455`).
- Add a password (the tool has been tested with enabled Authentication)

### Tool Configuration

- Update the `DEFAULT` variables in the `index.html` file:
  ```javascript
  const DEFAULT_OBS_HOST = "ws://localhost:4455"; // Replace with your OBS WebSocket URL
  const DEFAULT_OBS_PASSWORD = "1234567890"; // Replace with your WebSocket password
  ```

---

## Usage

- Open the **Transformation Tool** dock in OBS.
- Select the **Scene Name** from dropdown list and the **Source Name** from a dropdown list based on the selected scene.
- Adjust the **Position X**, **Position Y**, **Scale**, **Rotation**, and crop values (**Left**, **Top**, **Right**, **Bottom**).
- The tool fetches the existing values on page load and updates the input fields.
- Any changes to the inputs are immediately sent to OBS and applied in real-time.
- Refresh button syncs any manual transforms from OBS to input fields.

### Save Buttons (S1, S2, S3, S4):

- Function: Save the current values from the input fields to localStorage.
  
- Keys:
  - S1 saves to DTT1
  - S2 saves to DTT2
  - S3 saves to DTT3
  - S4 saves to DTT4

- Stored Data:
  - sceneName: Name of the OBS scene.
  - dataSourceName: Name of the source in the scene.
  - positionX, positionY: Position of the item in the scene.
  - scale: Scaling factor of the item.
  - rotation: Rotation of the item in degrees.
  - cropLeft, cropTop, cropRight, cropBottom: Cropping parameters for the item.
  - sceneItemId: Scene item ID (data source, group)

### Retrieve Buttons (T1, T2, T3, T4):
- Function: Retrieve previously saved values from localStorage and apply the transformation to the OBS scene item.
  
- Keys:
  - T1 retrieves from DTT1
  - T2 retrieves from DTT2
  - T3 retrieves from DTT3
  - T4 retrieves from DTT4
  
- Behavior:
  - Populates the input fields with the retrieved values.
  - Automatically sends a WebSocket message to OBS to apply the transformation.

---

A sample of the dock web page is presented below:

<img src="https://github.com/ManolisMariakakis/OBS-Dynamic-Transformation-Tool/blob/main/dock.png"/>

---

## WebSocket Commands and Authentication

The OBS WebSocket commands and the authentication process can also be utilized in Postman for testing and debugging. The commands are available in the following page:

[OBS WebSocket Commands](https://github.com/ManolisMariakakis/OBS-Dynamic-Transformation-Tool/blob/main/OBS_WebSocket_Commands_README.md)

Enjoy

