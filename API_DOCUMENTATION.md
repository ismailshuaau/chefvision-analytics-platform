# Voice Analytics Platform - API Documentation

## Analysis API (/api/analysis/)

### Overview
The Analysis API provides AI-powered analysis capabilities for transcribed content. Analysis jobs are processed asynchronously using Celery workers and Azure OpenAI services.

### Base URL
```
http://127.0.0.1:8000/api/analysis/
```

### Authentication
All endpoints require JWT authentication via `Authorization: Bearer <token>` header.

---

## Endpoints

### 1. CREATE Analysis Job
**POST /api/analysis/**

Creates a new analysis job and queues it for processing.

**Request Body:**
```json
{
  "transcription_session": 1,
  "analysis_type": "summary",
  "provider": "openai",
  "model_used": "gpt-4o",
  "custom_prompt": "Optional custom prompt for custom_analysis"
}
```

**Fields:**
- `transcription_session` (required): ID of completed transcription session
- `analysis_type` (required): One of: "summary", "key_points", "action_items", "sentiment", "custom_analysis"
- `provider` (optional): AI provider, defaults to "openai"
- `model_used` (optional): Model name, defaults to "gpt-4o"
- `custom_prompt` (required for custom_analysis): Custom analysis prompt

**Response (201 Created):**
```json
{
  "id": "acaf2042-b8bf-447b-8135-6f3b3d6f12b9",
  "transcription_session": 1,
  "transcription_title": "Meeting 17:32:53",
  "provider": "openai",
  "model_used": "gpt-4o",
  "analysis_type": "summary",
  "custom_prompt": null,
  "job_status": "queued",
  "provider_job_id": null,
  "input_tokens": null,
  "output_tokens": null,
  "input_text_length": null,
  "estimated_cost": null,
  "retry_count": 0,
  "error_message": null,
  "processing_started_at": null,
  "processing_completed_at": null,
  "created_at": "2025-09-26T10:33:01.223417Z",
  "processing_time": null,
  "total_tokens": null,
  "results": []
}
```

### 2. GET Analysis Job
**GET /api/analysis/{id}/**

Retrieves details of a specific analysis job.

**Response (200 OK):**
```json
{
  "id": "acaf2042-b8bf-447b-8135-6f3b3d6f12b9",
  "transcription_session": 139,
  "transcription_title": "Meeting 17:32:53",
  "provider": "openai",
  "model_used": "gpt-4o",
  "analysis_type": "summary",
  "custom_prompt": null,
  "job_status": "completed",
  "provider_job_id": null,
  "input_tokens": 40,
  "output_tokens": 20,
  "input_text_length": null,
  "estimated_cost": "0.0024",
  "retry_count": 0,
  "error_message": null,
  "processing_started_at": "2025-09-26T10:33:01.265316Z",
  "processing_completed_at": "2025-09-26T10:33:05.096247Z",
  "created_at": "2025-09-26T10:33:01.223417Z",
  "processing_time": 3.830931,
  "total_tokens": 60,
  "results": [
    {
      "id": 1,
      "analysis_type": "summary",
      "result_format": "text",
      "result_content": "The transcription appears to only contain the sequence of numbers from 1 to 10 in order.",
      "confidence_score": null,
      "tokens_used": null,
      "model_response_time_ms": 3813,
      "created_at": "2025-09-26T10:36:00.355641Z"
    }
  ]
}
```

### 3. LIST Analysis Jobs
**GET /api/analysis/**

Lists all analysis jobs for the authenticated user's transcriptions.

**Query Parameters:**
- `job_status`: Filter by status (queued, processing, completed, failed)
- `analysis_type`: Filter by type
- `transcription_session`: Filter by session ID

**Response (200 OK):**
```json
{
  "count": 13,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": "job-uuid",
      "transcription_session": 1,
      "transcription_title": "Meeting Title",
      "job_status": "completed",
      "analysis_type": "summary",
      "created_at": "2025-09-26T10:33:01.223417Z",
      "results": [...]
    }
  ]
}
```

### 4. GET Analysis Result
**GET /api/analysis/{id}/result/**

Get the analysis results for a completed job.

**Response (200 OK):** (Single result)
```json
{
  "id": 1,
  "analysis_type": "summary",
  "result_format": "text",
  "result_content": "The analysis content here...",
  "confidence_score": 0.95,
  "tokens_used": 150,
  "model_response_time_ms": 3813,
  "created_at": "2025-09-26T10:36:00.355641Z"
}
```

### 5. RETRY Analysis Job
**POST /api/analysis/{id}/retry/**

Retries a failed or completed analysis job.

**Response (200 OK):**
```json
{
  "id": "job-uuid",
  "job_status": "queued",
  "error_message": null,
  "processing_started_at": null,
  "processing_completed_at": null,
  "results": []
}
```

### 6. GET Jobs by Session
**GET /api/analysis/by_session/?session_id={id}**

Get all analysis jobs for a specific transcription session.

**Response (200 OK):**
```json
[
  {
    "id": "job-uuid",
    "analysis_type": "summary",
    "job_status": "completed",
    "results": [...]
  }
]
```

### 7. GET Status Summary
**GET /api/analysis/status_summary/**

Get summary of job statuses for the user.

**Response (200 OK):**
```json
{
  "queued": 2,
  "processing": 1,
  "completed": 10,
  "failed": 0,
  "total": 13,
  "recent_jobs": [...]
}
```

### 8. BULK Analysis
**POST /api/analysis/bulk_analyze/**

Create hierarchical bulk analysis based on user role and scope.

**Request Body:**
```json
{
  "scope": "personal",
  "date_from": "2025-01-01",
  "date_to": "2025-01-31",
  "template_id": "template-uuid",
  "provider": "openai",
  "model": "gpt-4o",
  "include_metrics": true
}
```

---

## Data Models

### AIAnalysisJob
- `id`: UUID (Primary Key)
- `transcription_session`: ForeignKey to TranscriptionSession
- `provider`: Choice field (openai, azure_text_analytics, deepseek, gpt-5)
- `model_used`: Choice field (gpt-4, gpt-4o, gpt-3.5-turbo, gpt-5, deepseek-chat)
- `analysis_type`: Choice field (sentiment, summary, key_points, action_items, custom_analysis)
- `job_status`: Choice field (pending, queued, processing, completed, failed)
- `custom_prompt`: TextField (optional)
- `input_tokens`, `output_tokens`: IntegerField
- `estimated_cost`: DecimalField
- `processing_started_at`, `processing_completed_at`: DateTimeField
- `results`: Related LLMAnalysisResult objects

### LLMAnalysisResult
- `id`: AutoField (Primary Key)
- `analysis_job`: ForeignKey to AIAnalysisJob
- `analysis_type`: Choice field (sentiment, summary, key_points, action_items, custom)
- `result_format`: Choice field (text, json, structured)
- `result_content`: JSONField (contains the actual analysis content)
- `confidence_score`: FloatField (0.0-1.0)
- `tokens_used`: IntegerField
- `model_response_time_ms`: IntegerField

---

## Job Status Flow

1. **pending** → Job created but not yet queued
2. **queued** → Job queued for processing
3. **processing** → Job being processed by Celery worker
4. **completed** → Job finished successfully (results available)
5. **failed** → Job failed (check error_message)

---

## Error Responses

### 400 Bad Request
```json
{
  "error": "Transcription must be completed before analysis."
}
```

### 404 Not Found
```json
{
  "error": "Transcription session not found"
}
```

### 403 Forbidden
```json
{
  "error": "You can only analyze your own transcriptions."
}
```

---

## Frontend Integration

### Correct Result Parsing
When a job completes, the analysis content is available in:
```javascript
result.results[0].result_content  // String containing the analysis
```

### Example Usage
```javascript
// Create analysis job
const analysisRequest = {
  transcription_session: sessionId,
  analysis_type: 'summary',
  provider: 'openai',
  model_used: 'gpt-4o'
};

// Poll for completion and extract results
onComplete: (result) => {
  if (result.results?.length > 0) {
    const analysisContent = result.results[0].result_content;
    // Display analysisContent to user
  }
}
```

---

## Backend Architecture

### Celery Task Processing
- Analysis jobs are processed asynchronously by `process_analysis_job` task
- Tasks use `OpenAIService` class for Azure OpenAI integration
- Results are saved to `LLMAnalysisResult` model (not `AIAnalysisResult`)
- Token usage and costs are tracked and calculated

### Model Relationship
```
AIAnalysisJob (1) → (Many) LLMAnalysisResult
AIAnalysisJob.results.all()  # Returns LLMAnalysisResult QuerySet
```

### Key Services
- `OpenAIService`: Handles Azure OpenAI API calls
- `process_analysis_job`: Main Celery task for processing
- `AIAnalysisJobViewSet`: REST API endpoints

---

## Fixed Issues

1. **Backend Task Model Mismatch**: Fixed task to save results to `LLMAnalysisResult` instead of `AIAnalysisResult`
2. **Empty Results Array**: Migrated existing results to correct model
3. **Frontend Parsing**: Updated to correctly extract `result.results[0].result_content`
4. **API Consistency**: Ensured serializers and tasks use same data models