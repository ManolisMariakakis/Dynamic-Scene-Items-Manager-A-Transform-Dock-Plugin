
# OBS WebSocket Commands and Authentication

This document details the commands used to interact with the OBS WebSocket, the responses they generate, and the process to calculate the authentication string. Ensure you are working with OBS WebSocket protocol V5 or higher, as the structure (e.g., op, d) aligns with this version.

Ensure placeholders like <scene_name> and <source_name_number> are dynamically replaced during runtime.

---

## Commands and Responses

### 1. **GetSceneList**
- **Command**:
  ```json
  {
    "op": 6,
    "d": {
      "requestType": "GetSceneList",
      "requestId": "getScenes"
    }
  }
  ```

- **Response**:
  ```json
  {
    "op": 7,
    "d": {
      "requestId": "getScenes",
      "requestStatus": {
        "result": true,
        "code": 100
      },
      "responseData": {
        "currentProgramSceneName": "Scene 1",
        "currentPreviewSceneName": "Scene 2",
        "scenes": [
          {
            "sceneIndex": 0,
            "sceneName": "Scene 1"
          },
          {
            "sceneIndex": 1,
            "sceneName": "Scene 2"
          }
        ]
      }
    }
  }
  ```

---

### 2. **GetSceneItemList**
- **Command**:
  ```json
  {
    "op": 6,
    "d": {
      "requestType": "GetSceneItemList",
      "requestId": "fetchSceneItemId",
      "requestData": {
        "sceneName": "<scene_name>"
      }
    }
  }
  ```

- **Response**:
  ```json
  {
    "op": 7,
    "d": {
      "requestId": "fetchSceneItemId",
      "requestStatus": {
        "result": true,
        "code": 100
      },
      "responseData": {
        "sceneItems": [
          {
            "sceneItemId": "<source_name_number>",
            "sourceName": "<source_name>",
            "sceneItemEnabled": true
          },
          {
            "sceneItemId": "<source_name_number>",
            "sourceName": "<source_name>",
            "sceneItemEnabled": true
          }
        ]
      }
    }
  }
  ```

---

### 3. **GetSceneItemTransform**
- **Command**:
  ```json
  {
    "op": 6,
    "d": {
      "requestType": "GetSceneItemTransform",
      "requestId": "fetchTransformValues",
      "requestData": {
        "sceneName": "<scene_name>",
        "sceneItemId": "<source_name_number>"
      }
    }
  }
  ```

- **Response**:
  ```json
  {
    "op": 7,
    "d": {
      "requestId": "fetchTransformValues",
      "requestStatus": {
        "result": true,
        "code": 100
      },
      "responseData": {
        "sceneItemTransform": {
          "positionX": 100,
          "positionY": 200,
          "scaleX": 1.0,
          "scaleY": 1.0,
          "rotation": 0,
          "cropLeft": 10,
          "cropTop": 5,
          "cropRight": 0,
          "cropBottom": 0
        }
      }
    }
  }
  ```

---

### 4. **SetSceneItemTransform**
- **Command**:
  ```json
  {
    "op": 6,
    "d": {
      "requestType": "SetSceneItemTransform",
      "requestId": "applyTransform",
      "requestData": {
        "sceneName": "<scene_name>",
        "sceneItemId": "<source_name_number>",
        "sceneItemTransform": {
          "positionX": 150,
          "positionY": 250,
          "scaleX": 1.0,
          "scaleY": 1.0,
          "rotation": 45,
          "cropLeft": 0,
          "cropTop": 0,
          "cropRight": 5,
          "cropBottom": 10
        }
      }
    }
  }
  ```

- **Response**:
  ```json
  {
    "op": 7,
    "d": {
      "requestId": "applyTransform",
      "requestStatus": {
        "result": true,
        "code": 100
      }
    }
  }
  ```

---

### 5. **Authenticate WebSocket Connection**
- **Command**:
  ```json
  {
    "op": 1,
    "d": {
      "rpcVersion": 1,
      "authentication": "<authentication_string>"
    }
  }
  ```

- **Response**:
  ```json
  {
    "op": 2,
    "d": {
      "negotiatedRpcVersion": 1
    }
  }
  ```

  ### Authentication Process

    The `authentication` string is calculated as follows:

    ### 1. Receive the Salt and Challenge
    OBS WebSocket sends a "Hello" message containing:
    - `salt`: Random string used for password hashing.
    - `challenge`: Random string used for final authentication.

    Example:
    ```json
    {
      "op": 0,
      "d": {
        "authentication": {
          "challenge": "c6f8c1...",
          "salt": "e3b9a4..."
        },
        "rpcVersion": 1
      }
    }
    ```

    ### 2. Calculate Authentication String
    Steps:
    1. Combine password with salt and hash:
       ```plaintext
       hashedPasswordSalt = SHA256(password + salt)
       ```
    
    2. Encode the hash in Base64:
       ```plaintext
       encodedPasswordSalt = Base64(hashedPasswordSalt)
       ```
    
    3. Combine the encoded hash with the challenge and hash again:
       ```plaintext
       finalHash = SHA256(encodedPasswordSalt + challenge)
       ```
    
    4. Encode the final hash in Base64:
       ```plaintext
       authentication = Base64(finalHash)
       ```

    ### JavaScript Implementation
    ```javascript
    async function calculateAuthentication(password, salt, challenge) {
        const encodeSHA256 = async (input) => {
            const encoder = new TextEncoder();
            const data = encoder.encode(input);
            const hashBuffer = await crypto.subtle.digest('SHA-256', data);
            return Array.from(new Uint8Array(hashBuffer))
                .map(byte => byte.toString(16).padStart(2, '0'))
                .join('');
        };
    
        const toBase64 = (hexString) => {
            const bytes = hexString.match(/.{2}/g).map(byte => parseInt(byte, 16));
            return btoa(String.fromCharCode(...bytes));
        };
    
        const hashedPasswordSalt = await encodeSHA256(password + salt);
        const encodedPasswordSalt = toBase64(hashedPasswordSalt);
        const combinedWithChallenge = encodedPasswordSalt + challenge;
        const finalHash = await encodeSHA256(combinedWithChallenge);
        return toBase64(finalHash);
    }
    ```
    
    ---
    
    ### Example Usage
    Given:
    - `password = "1234567890"`
    - `salt = "e3b9a4..."`
    - `challenge = "c6f8c1..."`
    
    Run:
    ```javascript
    calculateAuthentication("1234567890", "e3b9a4...", "c6f8c1...")
        .then(authentication => console.log("Authentication:", authentication));
    ```
---

### 5. **Connect to OBS WebSocket (ws://localhost:4455)**
- **Command**:
  The connection is initiated using the WebSocket URL `ws://localhost:4455`.

- **Code Example**:
  ```javascript
  const obs = new WebSocket("ws://localhost:4455");

  obs.onopen = () => {
      console.log("Connected to OBS WebSocket at ws://localhost:4455");
  };

  obs.onmessage = (event) => {
      const response = JSON.parse(event.data);
      console.log("WebSocket Response:", response);
  };

  obs.onclose = (event) => {
      console.log(`WebSocket closed: Code=${event.code}, Reason=${event.reason}`);
  };

  obs.onerror = (error) => {
      console.error("WebSocket error:", error);
  };
  ```

- **Explanation**:
  - This establishes a WebSocket connection to OBS WebSocket running locally on `ws://localhost:4455`.
  - The callbacks `onopen`, `onmessage`, `onclose`, and `onerror` handle the WebSocket events.
---
