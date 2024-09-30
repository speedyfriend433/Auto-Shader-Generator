# Shader Converter

Shader Converter is a simple command-line tool that allows you to convert ShaderToy shader links into `.fsh` files. By providing a ShaderToy link, the tool fetches the shader code using the ShaderToy API, processes it by replacing specific variables, and generates a `menu-shader.fsh` file.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Example](#example)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## Features

- **Shader Link Input**: Provide ShaderToy shader links to fetch corresponding shader code.
- **Automatic Processing**: Replaces specific ShaderToy variables with desired GLSL variables.
- **File Generation**: Generates a `menu-shader.fsh` file containing the processed shader code.
- **Error Handling**: Provides clear error messages for invalid links or failed requests.

## Prerequisites

Before using Shader Converter, ensure you have the following:

- **Node.js**: Version 12 or higher is recommended. [Download Node.js](https://nodejs.org/)
- **ShaderToy API Key**: Needed to access the ShaderToy API.

## Installation

### 1. Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/speedyfriend433/auto-shader-converter.git
cd auto-shader-converter
```

### 2. Install Dependencies

Install the necessary Node.js packages:

```bash
npm install
```

### 3. Obtain ShaderToy API Key

To interact with the ShaderToy API, you need an API key. Follow these steps to obtain one:

1. **Create or Log In to Your ShaderToy Account**:
   - Visit [ShaderToy](https://www.shadertoy.com/) and create an account or log in if you already have one.

2. **Request API Access**:
   - Navigate to your account settings or the developer section.
   - Look for options to request an API key. If not directly available, contact ShaderToy support for assistance.

3. **Store Your API Key**:
   - Once you receive your API key, keep it secure. You will need it in the next steps.

## Usage

### 1. Configure the API Key

Create a `.env` file in the root directory and add your ShaderToy API key:

```bash
touch .env
```

Open the `.env` file in a text editor and add the following line:

```env
SHADERTOY_API_KEY=YOUR_SHADERTOY_API_KEY_HERE
```

*Replace `YOUR_SHADERTOY_API_KEY_HERE` with your actual ShaderToy API key.*

### 2. Run the Shader Converter

Use the following command to convert a ShaderToy link into a `.fsh` file:

```bash
node shader-converter.js "https://www.shadertoy.com/view/mtyGWy"
```

*Replace the URL with your desired ShaderToy shader link.*

### 3. Output

After running the command, a `menu-shader.fsh` file will be generated in the current directory containing the processed shader code.

## Configuration

The `shader-converter.js` script processes the shader code by replacing specific ShaderToy variables with desired GLSL variables and adding uniform declarations. You can modify the processing rules within the script if needed.

## Example

### Converting a ShaderToy Link

1. **Command:**

   ```bash
   node shader-converter.js "https://www.shadertoy.com/view/mtyGWy"
   ```

2. **Output:**

   A `menu-shader.fsh` file is created with the processed shader code.

### Sample `menu-shader.fsh` Content

```glsl
uniform vec2 center; 
uniform vec2 resolution;
uniform float time;
uniform vec2 mouse; 
uniform float pulse1;
uniform float pulse2;
uniform float pulse3; 


// Processed shader code starts here
void main(){
    // Your shader code...
}
```

## Troubleshooting

### 1. "Load failed" Error

**Cause:**
- The ShaderToy API key is missing or incorrect.
- The shader ID extracted from the link is invalid.
- Network issues preventing API access.

**Solutions:**
- Ensure you have correctly set your ShaderToy API key in the `.env` file.
- Verify that the shader link is valid and follows the format `https://www.shadertoy.com/view/SHADER_ID`.
- Check your internet connection.

### 2. API Rate Limiting

**Cause:**
- Exceeded the number of allowed API requests within a given timeframe.

**Solutions:**
- Check ShaderToy API documentation for rate limits.
- Implement request throttling or limit the number of conversions.

### 3. Missing Dependencies

**Cause:**
- Required Node.js packages are not installed.

**Solutions:**
- Run `npm install` to install all dependencies.

### 4. File Permissions

**Cause:**
- Insufficient permissions to write files in the current directory.

**Solutions:**
- Ensure you have write permissions in the directory where you're running the script.
- Try running the command with elevated permissions.

## License

This project is licensed under the [MIT License](LICENSE).

---

**Note**: Always ensure you comply with ShaderToy's [Terms of Service](https://www.shadertoy.com/legal) when using their API and shaders. This tool is intended for personal and educational use. For commercial applications, obtain necessary permissions and licenses.

# Shader Converter Script

Below is the `shader-converter.js` script that performs the shader conversion. Ensure you have configured the `.env` file with your ShaderToy API key before running the script.

```javascript
// shader-converter.js

const fetch = require('node-fetch');
const fs = require('fs');
require('dotenv').config();

// Retrieve the ShaderToy API key from the environment variables
const API_KEY = process.env.SHADERTOY_API_KEY;

// Function to extract Shader ID from the URL
function extractShaderId(url) {
    const regex = /https?:\/\/www\.shadertoy\.com\/view\/([a-zA-Z0-9]+)/;
    const match = url.match(regex);
    return match ? match[1] : null;
}

// Function to process the shader code
function processShaderCode(rawText) {
    let processedText = rawText
        .replace(/iResolution/g, "resolution")
        .replace(/iTime/g, "time")
        .replace(/iMouse/g, "mouse")
        .replace(/fragCoord/g, "gl_FragCoord")
        .replace(/fragColor/g, "gl_FragColor")
        .replace(/void mainImage/g, "void main");

    // Add uniform declarations at the beginning
    processedText = `uniform vec2 center; 
uniform vec2 resolution;
uniform float time;
uniform vec2 mouse; 
uniform float pulse1;
uniform float pulse2;
uniform float pulse3; 


${processedText}`;

    return processedText;
}

// Function to fetch shader code from ShaderToy API
async function fetchShaderCode(shaderId) {
    if (!API_KEY || API_KEY === 'YOUR_SHADERTOY_API_KEY_HERE') {
        throw new Error("ShaderToy API key is not set. Please set it in the .env file.");
    }

    const apiUrl = `https://www.shadertoy.com/api/v1/shaders/${shaderId}?key=${API_KEY}`;

    const response = await fetch(apiUrl);
    if (!response.ok) {
        throw new Error(`Unable to fetch shader. HTTP Status: ${response.status}`);
    }

    const data = await response.json();
    if (data && data.Shader && data.Shader.Code) {
        return data.Shader.Code;
    } else {
        throw new Error("Shader code not found in the API response.");
    }
}

// Function to save processed shader code to a file
function saveShaderToFile(shaderCode, filename = 'menu-shader.fsh') {
    fs.writeFileSync(filename, shaderCode, 'utf8');
    console.log(`Shader successfully saved to ${filename}`);
}

// Main function to execute the conversion
async function main() {
    const args = process.argv.slice(2);
    if (args.length === 0) {
        console.error("Usage: node shader-converter.js <ShaderToy_Shader_Link>");
        process.exit(1);
    }

    const shaderLink = args[0];
    const shaderId = extractShaderId(shaderLink);

    if (!shaderId) {
        console.error("Invalid ShaderToy link. Please provide a link in the format: https://www.shadertoy.com/view/SHADER_ID");
        process.exit(1);
    }

    try {
        console.log(`Fetching shader with ID: ${shaderId}`);
        const rawShaderCode = await fetchShaderCode(shaderId);
        const processedShaderCode = processShaderCode(rawShaderCode);
        saveShaderToFile(processedShaderCode);
    } catch (error) {
        console.error(`Error: ${error.message}`);
        process.exit(1);
    }
}

main();
```

## Explanation of the Script

1. **Dependencies**:
   - `node-fetch`: To make HTTP requests to the ShaderToy API.
   - `fs`: To handle file system operations (writing the `.fsh` file).
   - `dotenv`: To manage environment variables securely.

2. **Environment Variables**:
   - The script uses the `dotenv` package to load the ShaderToy API key from a `.env` file. Ensure you have created the `.env` file with your API key as described in the [Installation](#installation) section.

3. **Functions**:
   - `extractShaderId(url)`: Extracts the shader ID from a given ShaderToy URL using a regular expression.
   - `processShaderCode(rawText)`: Processes the raw shader code by replacing specific variables and adding uniform declarations.
   - `fetchShaderCode(shaderId)`: Fetches the shader code from the ShaderToy API using the provided shader ID and API key.
   - `saveShaderToFile(shaderCode, filename)`: Saves the processed shader code to a `.fsh` file.

4. **Main Execution**:
   - The script accepts a ShaderToy link as a command-line argument.
   - It extracts the shader ID from the link.
   - Fetches the shader code using the ShaderToy API.
   - Processes the shader code.
   - Saves the processed code to `menu-shader.fsh`.

## Running the Script

1. **Ensure Dependencies are Installed**:

   ```bash
   npm install
   ```

2. **Set Your ShaderToy API Key**:

   - Open the `.env` file and set your API key:

     ```env
     SHADERTOY_API_KEY=YOUR_SHADERTOY_API_KEY_HERE
     ```

3. **Run the Script with a ShaderToy Link**:

   ```bash
   node shader-converter.js "https://www.shadertoy.com/view/mtyGWy"
   ```

   Replace the URL with your desired ShaderToy shader link.

4. **Result**:

   - Upon successful execution, a `menu-shader.fsh` file will be created in the current directory containing the processed shader code.

## Notes

- **API Key Security**: Ensure that your `.env` file is not exposed publicly, especially if sharing your code repository. Add `.env` to your `.gitignore` file to prevent accidental commits.

- **Error Handling**: The script includes basic error handling to inform you if something goes wrong, such as an invalid link or issues with the API request.

- **Customization**: You can modify the `processShaderCode` function to include additional processing rules as needed.

## License

This project is licensed under the [MIT License](LICENSE).

---

Feel free to contribute to this project by opening issues or submitting pull requests. For any questions or support, please contact [speedyfriend433@gmail.com](mailto:speedyfriend433@gmail.com).
