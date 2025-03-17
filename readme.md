# Lamatic SDK

The Lamatic SDK provides a simple way to interact with the Lamatic AI platform, allowing you to execute workflows and integrate AI capabilities into your applications.

## Installation

Install the Lamatic SDK using npm:

```bash
npm install lamatic-ai
```

## Getting Started

To use the Lamatic SDK, you'll need:
- An API key or access token
- Your project ID
- The Lamatic API endpoint

Initialize the SDK like this:

```typescript
import { Lamatic } from "lamatic-ai";

const lamatic = new Lamatic({
  apiKey: "your-api-key",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});
```

## Authentication

The SDK supports two authentication methods:

### API Key

```typescript
const lamatic = new Lamatic({
  apiKey: "your-api-key",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});
```

### Access Token

```typescript
const lamatic = new Lamatic({
  accessToken: "your-access-token",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});
```

You must provide either an API key or an access token, but not both.

## Executing Workflows

The main functionality of the SDK is to execute workflows (flows) that you've created on the Lamatic platform.

### Basic Example

```typescript
import { Lamatic } from "lamatic-ai";

const lamatic = new Lamatic({
  apiKey: "your-api-key",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});

async function main() {
  const flowId = "your-flow-id";
  // sample payload
  const payload = {
    prompt: "hey, how are you?"
  }
  try {
    const response = await lamatic.executeFlow(flowId, payload);
    console.log(response);
  } catch (error) {
    console.error(error);
  }
}

main();
```

### Using Access Token

```typescript
import { Lamatic } from "lamatic-ai";

const lamatic = new Lamatic({
  accessToken: "your-access-token",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});

async function main() {
  const flowId = "your-flow-id";
  // sample payload
  const payload = {
    prompt: "hey, how are you?"
  }
  const response = await lamatic.executeFlow(flowId, payload);
  console.log(response);
}

main();
```

### Handling Expired Access Tokens

This example shows how to handle an expired access token by fetching a new one and updating the SDK:

```typescript
import { Lamatic } from "lamatic-ai";

const lamatic = new Lamatic({
  accessToken: "your-access-token",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});

async function getAccessToken() {
  const response = await fetch("your-server-endpoint/get-access-token", {
    method: "GET",
  });
  const data = await response.json();
  return data.accessToken;
}

async function main() {
  const flowId = "your-flow-id";
  // sample payload
  const payload = {
    prompt: "hey, how are you?"
  }
  const response = await lamatic.executeFlow(flowId, payload);
  console.log(response);
  if (response.statusCode === 403) {

    // Get the new access token
    const accessToken = await getAccessToken();
    // Update the access token for the Lamatic SDK
    lamatic.updateAccessToken(accessToken);

    // Execute the flow again
    const response = await lamatic.executeFlow(flowId, payload);
    console.log(response);
  }
}

main();
```

Here, you are supposed to have a server endpoint that returns a new access token when called. The server should handle the authentication and return the new access token from the same secret generated from Lamatic Studio.

### Response Structure

The `executeFlow` method returns a promise that resolves to a `LamaticResponse` object with the following structure:

```typescript
{
  status: "success" | "error",  // Status of the workflow execution
  result: object | null,        // The result data from the workflow (if successful)
  message?: string,             // Error message (if status is "error")
  statusCode?: number           // HTTP status code
}
```

### Example with Error Handling

```typescript
async function runWorkflow() {
  const flowId = "your-flow-id";
  const payload = {
    input: "Generate a creative story about robots",
    parameters: {
      temperature: 0.7,
      maxTokens: 500
    }
  };
  
  try {
    const response = await lamatic.executeFlow(flowId, payload);
    
    if (response.status === "success") {
      console.log("Workflow executed successfully!");
      console.log("Result:", response.result);
      return response.result;
    } else {
      console.error("Workflow execution failed:", response.message);
      return null;
    }
  } catch (error) {
    console.error("Exception occurred:", error);
    return null;
  }
}
```

## Updating Access Token

If you're using access token authentication, you can update the token during runtime, if the need be:

```typescript
// Initialize with an access token
const lamatic = new Lamatic({
  accessToken: "initial-access-token",
  projectId: "your-project-id",
  endpoint: "your-endpoint",
});

// Later, update the access token
lamatic.updateAccessToken("new-access-token");
```

## Advanced Usage

### Customizing Payloads

The payload structure depends on the specific workflow you're executing. Make sure to check the documentation for your workflow to understand what parameters it expects.

```typescript
// Example for a text generation workflow
const textGenPayload = {
  prompt: "Write a poem about artificial intelligence",
  options: {
    temperature: 0.8,
    topP: 0.95,
    maxLength: 200
  }
};

// Example for an image analysis workflow
const imageAnalysisPayload = {
  imageUrl: "https://example.com/image.jpg",
  analysisTypes: ["objects", "faces", "text"]
};
```

### Handling Responses

Different workflows may return different result structures. Here's how you might handle responses from different types of workflows:

```typescript
// Text generation workflow response
const textResponse = await lamatic.executeFlow("text-gen-flow-id", textGenPayload);
if (textResponse.status === "success" && textResponse.result) {
  const generatedText = textResponse.result.text;
  console.log("Generated text:", generatedText);
}

// Classification workflow response
const classificationResponse = await lamatic.executeFlow("classify-flow-id", classifyPayload);
if (classificationResponse.status === "success" && classificationResponse.result) {
  const category = classificationResponse.result.category;
  const confidence = classificationResponse.result.confidence;
  console.log(`Classification: ${category} (confidence: ${confidence})`);
}
```

## TypeScript Support

The SDK is written in TypeScript and provides type definitions for all its functionality:

```typescript
import { Lamatic, LamaticResponse, LamaticConfig } from "lamatic-ai";

// Custom payload type
interface TextGenerationPayload {
  prompt: string;
  maxTokens?: number;
  temperature?: number;
}

// Custom response type
interface TextGenerationResult {
  text: string;
  usedTokens: number;
}

async function generateText(prompt: string): Promise<TextGenerationResult | null> {
  const payload: TextGenerationPayload = {
    prompt,
    maxTokens: 1000,
    temperature: 0.7
  };
  
  const response = await lamatic.executeFlow<TextGenerationResult>("text-gen-flow", payload);
  
  if (response.status === "success" && response.result) {
    return response.result;
  }
  
  return null;
}
```

## Error Handling

The SDK throws errors in case of configuration issues or network problems. It's recommended to always wrap SDK calls in try-catch blocks:

```typescript
try {
  const response = await lamatic.executeFlow(flowId, payload);
  // Process response
} catch (error) {
  if (error instanceof Error) {
    // Log the error
    console.error("SDK Error:", error.message);
    
    // Handle specific error scenarios
    if (error.message.includes("timeout")) {
      // Handle timeout
    } else if (error.message.includes("network")) {
      // Handle network error
    }
  }
}
```

## Environment Variables

For better security, consider using environment variables for sensitive information:

```typescript
// Using environment variables
const lamatic = new Lamatic({
  apiKey: process.env.LAMATIC_API_KEY,
  projectId: process.env.LAMATIC_PROJECT_ID,
  endpoint: process.env.LAMATIC_ENDPOINT,
});
```

## Debugging

To help with debugging, the SDK logs errors to the console. You can capture these logs for troubleshooting purposes.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

# Support
For help and support, visit our documentation or contact support@lamatic.ai.