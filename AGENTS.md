# AGENTS.md

## Project Overview

**SaaS AI Business Idea Generator** is a full-stack application that leverages AI to generate innovative business ideas for AI agents in real-time. The application demonstrates streaming AI responses using Server-Sent Events (SSE) for a smooth user experience.

## Architecture

### Tech Stack

- **Frontend**: Next.js 15.5.6 (React 19.1.0)
- **Backend**: FastAPI (Python)
- **AI Model**: OpenAI GPT-5-nano
- **Styling**: Tailwind CSS 4
- **Streaming**: Server-Sent Events (SSE)

### System Components

```
┌─────────────────┐         ┌──────────────┐         ┌─────────────┐
│   Next.js UI    │ ──SSE──>│  FastAPI     │ ──API──>│   OpenAI    │
│   (Frontend)    │ <──────│  (Backend)   │ <──────│   GPT-5     │
└─────────────────┘         └──────────────┘         └─────────────┘
```

## Key Features

### 1. Real-Time AI Streaming
- Uses Server-Sent Events (SSE) for real-time streaming of AI-generated content
- Progressive rendering as tokens arrive from the OpenAI API
- Smooth user experience without waiting for complete response

### 2. Markdown Rendering
- Full markdown support with `react-markdown`
- GitHub Flavored Markdown (GFM) support via `remark-gfm`
- Automatic line breaks with `remark-breaks`
- Styled with Tailwind Typography

### 3. Modern UI/UX
- Gradient background design
- Responsive layout
- Dark mode support
- Loading states and animations
- Clean, professional card-based design

## API Documentation

### Endpoint: `/api`

**Method**: GET

**Description**: Generates a new AI agent business idea with streaming response.

**Response Type**: `text/event-stream`

**Request Flow**:
1. Client initiates EventSource connection to `/api`
2. Backend creates OpenAI streaming completion
3. Backend streams chunks as SSE events
4. Frontend accumulates and renders markdown in real-time

**Sample Response Format**:
```
data: # Business Idea
data: 
data: ## AI-Powered Personal Assistant
data: 
data: - Smart scheduling
data: - Email management
```

## Code Structure

### Backend (`/api/index.py`)

```python
@app.get("/api")
def idea():
    # Initialize OpenAI client
    client = OpenAI()
    
    # Prompt engineering for structured output
    prompt = [{
        "role": "user", 
        "content": "Reply with a new business idea for AI Agents, formatted with headings, sub-headings and bullet points"
    }]
    
    # Stream completion
    stream = client.chat.completions.create(
        model="gpt-5-nano", 
        messages=prompt, 
        stream=True
    )
    
    # Convert to SSE format
    def event_stream():
        for chunk in stream:
            text = chunk.choices[0].delta.content
            if text:
                lines = text.split("\n")
                for line in lines:
                    yield f"data: {line}\n"
                yield "\n"
    
    return StreamingResponse(event_stream(), media_type="text/event-stream")
```

### Frontend (`/pages/index.tsx`)

```typescript
const [idea, setIdea] = useState<string>('…loading');

useEffect(() => {
    const evt = new EventSource('/api');
    let buffer = '';

    evt.onmessage = (e) => {
        buffer += e.data;
        setIdea(buffer);
    };
    
    evt.onerror = () => {
        console.error('SSE error, closing');
        evt.close();
    };

    return () => { evt.close(); };
}, []);
```

## Environment Setup

### Required Environment Variables

Create a `.env` file in the root directory:

```env
OPENAI_API_KEY=your_openai_api_key_here
```

### Python Dependencies (`/api/requirements.txt`)

```
fastapi
openai
uvicorn
```

### Node Dependencies (key packages)

```json
{
  "dependencies": {
    "@tailwindcss/typography": "^0.5.19",
    "next": "15.5.6",
    "react": "19.1.0",
    "react-dom": "19.1.0",
    "react-markdown": "^10.1.0",
    "remark-breaks": "^4.0.0",
    "remark-gfm": "^4.0.1"
  }
}
```

## Development Workflow

### Local Development

1. **Install dependencies**:
   ```bash
   npm install
   pip install -r api/requirements.txt
   ```

2. **Set up environment**:
   ```bash
   export OPENAI_API_KEY=your_key_here
   ```

3. **Run backend**:
   ```bash
   uvicorn api.index:app --reload
   ```

4. **Run frontend**:
   ```bash
   npm run dev
   ```

5. **Access application**:
   ```
   http://localhost:3000
   ```

## AI Agent Patterns Demonstrated

### 1. Streaming Completions
- Real-time token streaming from LLM
- Progressive UI updates
- Better UX for long-form content generation

### 2. Prompt Engineering
- Structured output formatting
- Context-specific generation
- Markdown-formatted responses

### 3. Event-Driven Architecture
- Server-Sent Events for unidirectional streaming
- Efficient resource usage
- Automatic reconnection handling

### 4. State Management
- Client-side buffer accumulation
- React hooks for reactive updates
- Cleanup on component unmount

## Deployment Considerations

### Vercel Deployment (Recommended)

1. **Frontend**: Automatically deploys Next.js app
2. **Backend**: Deploy as serverless function
3. **Environment Variables**: Set `OPENAI_API_KEY` in Vercel dashboard

### Alternative Deployment Options

- **Docker**: Containerize both frontend and backend
- **Railway/Render**: Full-stack deployment with Python support
- **AWS Lambda**: Serverless backend with CloudFront frontend

## Future Enhancements

### Potential Features
- [ ] User authentication and saved ideas
- [ ] Idea history and favorites
- [ ] Multiple AI models (Claude, Gemini, etc.)
- [ ] Customizable prompts
- [ ] Idea refinement and iteration
- [ ] Export to PDF/Markdown
- [ ] Collaboration features
- [ ] Idea rating and feedback system
- [ ] Multi-language support
- [ ] Voice input for idea generation

### Performance Optimizations
- [ ] Response caching
- [ ] Rate limiting
- [ ] Error retry mechanisms
- [ ] Fallback models
- [ ] Progress indicators with ETA

## Security Best Practices

1. **API Key Management**: Never commit API keys
2. **Rate Limiting**: Implement request throttling
3. **Input Validation**: Sanitize all user inputs
4. **CORS Configuration**: Restrict allowed origins
5. **Error Handling**: Don't expose sensitive error details

## License

This project is part of the "AI in Production: Gen AI and Agentic AI at Scale" course.

## Contributing

Contributions are welcome! Please follow these guidelines:
1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## Support

For issues and questions, please open an issue on the GitHub repository.

---

**Repository**: https://github.com/Fakhruddin90/saas-ai-business-idea-generator
**Last Updated**: 2025-10-26
