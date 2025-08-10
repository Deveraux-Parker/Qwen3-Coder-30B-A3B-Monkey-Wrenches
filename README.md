<img width="1038" height="1597" alt="image" src="https://github.com/user-attachments/assets/4f3ac7ee-7b3f-46eb-93b3-c2d26f584350" />

## QWEN XML Tool Call Explorer

A powerful web-based interface for testing and exploring XML-formatted function calling with QWEN models through OpenAI-compatible APIs. This tool helps developers understand, test, and debug function calling capabilities with real-time visualization and interactive features.

## Features

### Core Functionality
- **Real-time XML Parsing**: Instantly parses and validates XML tool calls from model responses
- **Interactive Testing**: Test function calls with custom tools and see live results
- **Streaming Support**: Watch responses stream in real-time with token-by-token updates
- **Beautiful Visualization**: Clean, modern UI with syntax highlighting and formatted output

### Advanced Features
- **Custom Tool Builder**: Create and test your own function definitions
- **Tool Library**: Pre-built collection of common tools (calculator, web search, file operations, etc.)
- **Response Recovery**: Automatically attempts to fix malformed XML responses
- **Export/Import**: Save and share your tool configurations
- **API Key Support**: Works with both local VLLM servers and cloud API providers

## Getting Started

### Prerequisites
- A running VLLM server with a QWEN model, OR
- An OpenAI-compatible API endpoint with an API key
- A modern web browser (Chrome, Firefox, Safari, Edge)

### Installation

1. Clone or download the `tool_call_explorer.html` file
2. Open it directly in your web browser - no server setup required!

### Configuration

1. Click the **Settings** icon (gear icon in the top right)
2. Configure your API endpoint:
   - **For Local VLLM**: 
     - Server URL: `http://localhost`
     - Port: `8000` (or your VLLM port)
     - API Key: Leave blank
   - **For Cloud APIs**:
     - Server URL: Your API endpoint URL
     - Port: The API port (if applicable)
     - API Key: Your API key
3. Adjust generation parameters:
   - **Temperature**: Controls randomness (0.0-2.0)
   - **Top P**: Nucleus sampling threshold (0.0-1.0)
   - **Min P**: Minimum probability threshold (0.0-1.0)

## Usage Guide

### Basic Usage

1. **Enter a prompt** that requires tool usage:
   ```
   What's the weather like in San Francisco?
   ```

2. **Click "Generate Tool Calls"** to see how the model responds with XML-formatted function calls

3. **View the results** in multiple formats:
   - **Formatted View**: Visual representation of the function calls
   - **Raw XML**: The actual XML response from the model
   - **JSON View**: Parsed function calls in JSON format

### Working with Custom Tools

The Agentic loop will attempt to create a tool and validate it.

1. **Create a new tool**:
   - Click "Add Custom Tool"
   - Define your function with a clear description (don't be afraid to paste in documentation for an API or function you're calling)
   - Add parameters with types and descriptions
   - Save the tool for future use

2. **Example custom tool**:
   ```json
   {
     "name": "get_stock_price",
     "description": "Get the current stock price for a symbol",
     "parameters": {
       "type": "object",
       "properties": {
         "symbol": {
           "type": "string",
           "description": "The stock ticker symbol"
         }
       },
       "required": ["symbol"]
     }
   }
   ```

### Using the Tool Library

1. Browse pre-built tools in the library
2. Click any tool to add it to your active toolset
3. Remove tools by clicking the × on active tool badges

### Streaming Mode

Enable streaming to watch the model generate responses in real-time:
- Toggle "Enable Streaming" before generating
- See tokens appear as they're generated
- Stop generation at any time with the Stop button

## Understanding the Output

### XML Format
The tool expects responses in this format:
```xml
<tool_calls>
  <tool_call>
    <function>
      <name>function_name</name>
      <arguments>
        {"param": "value"}
      </arguments>
    </function>
  </tool_call>
</tool_calls>
```

### Status Indicators
- **Green**: Successful parsing and valid function calls
- **Yellow**: Partial success or recovery attempted
- **Red**: Parsing errors or invalid responses

## Tips and Best Practices

1. **Start with simple prompts** to test your setup
2. **Use the built-in tools** as examples for creating custom tools
3. **Enable streaming** for long responses to see progress
4. **Export successful tool configurations** for reuse
5. **Check the browser console** for detailed debugging information

## Troubleshooting

### Common Issues

**"Connection refused" error**
- Ensure your VLLM server is running
- Check the server URL and port in settings
- Verify firewall settings allow the connection

**"Unauthorized" error**
- Verify your API key is correct
- Check that the API endpoint supports OpenAI-compatible format

**No tool calls in response**
- Ensure your model supports function calling
- Try a more explicit prompt that clearly requires tool usage
- Check temperature settings (lower values are more deterministic)

**Malformed XML responses**
- The tool includes automatic recovery for common issues
- Try adjusting temperature and top_p for more consistent outputs
- Ensure your system prompt is properly formatted

## Technical Details

- **Framework**: Pure vanilla JavaScript with no dependencies
- **Compatibility**: Works with any OpenAI-compatible completion API
- **Storage**: Uses localStorage for settings and custom tools
- **Security**: API keys are stored locally and never transmitted except to your configured endpoint

## License

This tool is provided as-is for educational and development purposes. Feel free to modify and distribute according to your needs.

---

Built for the AI engineering community to make function calling development easier and more intuitive.
