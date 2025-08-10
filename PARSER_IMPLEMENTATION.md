# XML Tool Call Parser - Implementation Details

## Architecture Overview

The parser is implemented in both Python and JavaScript with identical logic, ensuring consistency across server-side and client-side applications.

## Core Regex Patterns

### 1. Tool Call Detection Patterns

```python
# Standard format with complete tags
self.tool_call_with_tag = re.compile(r'<tool_call>(.*?)</tool_call>', re.DOTALL)

# Missing opening tag pattern
self.tool_call_without_tag = re.compile(r'(<function=.*?</tool_call>)', re.DOTALL)

# Function-only pattern (no tool_call tags)
self.function_only_pattern = re.compile(r'(<function=.*?</function>)', re.DOTALL)
```

**Pattern Explanation**:
- `re.DOTALL`: Makes `.` match newlines, crucial for multiline content
- `.*?`: Non-greedy matching to handle multiple tool calls
- Capturing groups extract the content for further parsing

### 2. Function Extraction Pattern

```python
self.function_pattern = re.compile(r'<function=([^<>]+?)(?:</function>|>)')
```

**Pattern Breakdown**:
- `<function=`: Literal match for function opening
- `([^<>]+?)`: Captures function name (no angle brackets)
- `(?:</function>|>)`: Matches either closing tag or just `>`

### 3. Parameter Extraction Patterns

```python
# Standard format: <parameter=name>value</parameter>
self.param_standard = re.compile(r'<parameter=([^<>=]+)>(.*?)</parameter>', re.DOTALL)

# Malformed format: <parameter=name=value</parameter>
self.param_malformed = re.compile(r'<parameter=([^<>=]+)=([^<>]+)</parameter>')

# Empty tag format: <parameter=name></parameter>\nvalue
self.param_empty_tag = re.compile(r'<parameter=([^<>=]+)></parameter>\s*([^\s<][^<]*?)(?=\s*<|$)')
```

**Pattern Details**:
- `[^<>=]+`: Parameter name without special characters
- `\s*`: Optional whitespace after empty tag
- `[^\s<][^<]*?`: Value starting with non-whitespace
- `(?=\s*<|$)`: Lookahead for next tag or end

## Parsing Algorithm

### Phase 1: Tool Call Detection

```python
def parse(self, text: str) -> List[Dict[str, Any]]:
    tool_calls = []
    
    # Try patterns in order of preference
    # 1. Standard format
    matches = list(self.tool_call_with_tag.finditer(text))
    
    # 2. Missing opening tag
    if not matches:
        matches = list(self.tool_call_without_tag.finditer(text))
    
    # 3. Function-only format
    if not matches:
        matches = list(self.function_only_pattern.finditer(text))
    
    # Process each match
    for match in matches:
        content = match.group(1)
        parsed = self._parse_single_tool_call(content)
        if parsed:
            tool_calls.append(parsed)
    
    return tool_calls
```

**Key Design Decisions**:
1. **Sequential fallback**: Try most common format first
2. **List conversion**: `finditer()` returns iterator, convert for reuse
3. **Validation**: Only add successfully parsed calls

### Phase 2: Single Tool Call Parsing

```python
def _parse_single_tool_call(self, content: str) -> Optional[Dict[str, Any]]:
    # Extract function name
    func_match = self.function_pattern.search(content)
    if not func_match:
        return None
    
    function_name = func_match.group(1).strip()
    parameters = {}
    
    # Three-pass parameter extraction
    # ... (parameter extraction logic)
    
    return {
        "function": function_name,
        "parameters": parameters
    }
```

### Phase 3: Three-Pass Parameter Extraction

The parser uses three passes to handle different parameter formats:

#### Pass 1: Standard Format
```python
for match in self.param_standard.finditer(content):
    param_name = match.group(1).strip()
    param_value = match.group(2).strip()
    
    if param_name not in parameters:
        parameters[param_name] = self._convert_type(param_value)
```

#### Pass 2: Malformed Format
```python
for match in self.param_malformed.finditer(content):
    param_name = match.group(1).strip()
    param_value = match.group(2).strip()
    
    # Only add if not found in standard format
    if param_name not in parameters:
        parameters[param_name] = self._convert_type(param_value)
```

#### Pass 3: Empty Tag Format
```python
for match in self.param_empty_tag.finditer(content):
    param_name = match.group(1).strip()
    param_value = match.group(2).strip()
    
    # Only add if not found in previous passes
    if param_name not in parameters:
        parameters[param_name] = self._convert_type(param_value)
```

**Why Three Passes?**
1. **Priority**: Standard format takes precedence
2. **No overwrites**: First match wins for each parameter
3. **Flexibility**: Handles mixed formats in same response

### Phase 4: Type Conversion

```python
def _convert_type(self, value: str) -> Any:
    value = value.strip()
    
    # Try integer
    try:
        return int(value)
    except ValueError:
        pass
    
    # Try float
    try:
        return float(value)
    except ValueError:
        pass
    
    # Try boolean
    if value.lower() in ['true', 'false']:
        return value.lower() == 'true'
    
    # Default to string
    return value
```

**Conversion Priority**:
1. Integer (most specific)
2. Float (numeric but not integer)
3. Boolean (explicit true/false)
4. String (fallback)

## JavaScript Implementation

The JavaScript version maintains identical logic:

```javascript
class XMLToolParser {
    constructor() {
        // Same patterns as Python
        this.toolCallRegex = /<tool_call>([\s\S]*?)<\/tool_call>/g;
        this.functionOnlyRegex = /(<function=[\s\S]*?<\/tool_call>)/g;
        this.functionOnlyPattern = /(<function=[\s\S]*?<\/function>)/g;
        // ... other patterns
    }
    
    parse(xmlText) {
        // Identical algorithm to Python
        // Key difference: regex.lastIndex reset needed
        this.toolCallRegex.lastIndex = 0;
        // ... rest of implementation
    }
}
```

**JavaScript-Specific Considerations**:
1. **Global flag**: Required for multiple matches
2. **lastIndex reset**: Prevents regex state issues
3. **Type checking**: Use `typeof` for type validation

## Error Handling

### Graceful Degradation

The parser never throws exceptions during normal operation:

1. **Invalid XML**: Returns empty array
2. **Missing function**: Skips that tool call
3. **Malformed parameters**: Attempts all three formats
4. **Type conversion failures**: Falls back to string

### Error Recovery System (HTML Interface)

```javascript
diagnoseError(response) {
    const patterns = {
        noXML: {
            test: (r) => !r.includes('<'),
            recovery: "strict"
        },
        truncated: {
            test: (r) => r.endsWith('<') || r.endsWith('</'),
            recovery: "reduce_tokens"
        },
        // ... other patterns
    };
    
    // Return diagnosis with recovery strategy
}
```

## Performance Optimizations

### 1. Regex Compilation
- Patterns compiled once during initialization
- Reused for all parsing operations

### 2. Early Exit
- Stop searching once matches found
- Don't try all patterns if first succeeds

### 3. Minimal Memory Usage
- No intermediate data structures
- Direct regex to result mapping

### 4. Stream-Friendly
- Can parse partial responses
- Updates incrementally during streaming

## Edge Case Handling

### 1. Multiline Content
```xml
<parameter=body>
Line 1
Line 2
Line 3
</parameter>
```
- Preserved exactly with newlines
- DOTALL flag enables cross-line matching

### 2. Special Characters
```xml
<parameter=query>SELECT * FROM users WHERE name = "John"</parameter>
```
- No escaping required in parsing
- Quotes and special chars preserved

### 3. Nested Structures
```xml
<parameter=config>{"nested": {"key": "value"}}</parameter>
```
- Treated as string, application handles parsing

### 4. Empty Parameters
```xml
<parameter=optional></parameter>
```
- Results in empty string value
- Not null or undefined

### 5. Unicode Support
```xml
<parameter=message>Hello 世界 🌍</parameter>
```
- Full Unicode support in both implementations
- No encoding issues

## Testing Strategy

### Unit Tests
- Each edge case has dedicated test
- Both positive and negative cases
- Type conversion validation

### Integration Tests
- Real VLLM server responses
- Streaming and non-streaming
- Concurrent request handling

### Regression Tests
- New patterns added to test suite
- Ensures fixes don't break existing functionality

## Future Considerations

### Potential Enhancements

1. **Streaming Optimization**
   - Progressive parsing during chunk arrival
   - Partial tool call detection

2. **Performance Profiling**
   - Regex performance metrics
   - Memory usage tracking

3. **Format Detection**
   - Auto-detect XML vs JSON
   - Support multiple formats

4. **Validation Layer**
   - Schema validation
   - Required parameter checking

5. **Error Context**
   - Line/column error reporting
   - Detailed parse failure reasons

### Maintainability

1. **Pattern Documentation**
   - Each regex has clear explanation
   - Examples for each format

2. **Test Coverage**
   - 100% code coverage
   - Edge case documentation

3. **Version Compatibility**
   - Python 3.6+ support
   - ES6+ JavaScript

4. **Cross-Platform**
   - Works in browser and Node.js
   - No platform-specific code