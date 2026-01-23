Pipeline details

1. Reads survey questions from JSON files (one per country/language)
2. Sends each statement to the LLM multiple times (configurable)
3. Validates responses against allowed options
4. Logs results to CSV files
5. Stores full LLM responses as JSON files

## Project Structure

```
culture-alignment/
├── pipeline.ipynb          # Main pipeline notebook
├── requirements.txt        # Python dependencies
├── questions/              # Input survey files
│   ├── usa_english.json
│   └── alb_albanian.json
└── output/
    ├── generic/            # CSV results per file
    │   ├── usa_english.csv
    │   └── alb_albanian.csv
    └── responses/          # Full LLM response JSONs
        └── resp_*.json
```

## Input Format

Each question file in `questions/` follows this structure:

```json
{
    "id": "usa_english",
    "country": "United States",
    "language": "English",
    "questions": [
        {
            "id": "TQ_38_a_en-US",
            "description": "Definition of AI shown to respondent...",
            "responses": {
                "Strongly disagree": 1,
                "Disagree": 2,
                "Agree": 3,
                "Strongly agree": 4,
                "I don't know": 0
            },
            "questions": [
                {
                    "id": "TT4G35A",
                    "prompt": "Artificial intelligence helps teachers..."
                }
            ]
        }
    ]
}
```

- **language**: Language for the LLM to respond in
- **description**: AI definition provided to the LLM as context
- **responses**: Valid response options mapped to numeric scores
- **questions**: Array of statements with unique IDs and prompts

## Output Format

### CSV Output (`output/generic/*.csv`)

Each row represents one LLM response:

| Column | Description |
|--------|-------------|
| `question_id` | Statement ID (e.g., TT4G35A) |
| `run_number` | Which iteration (1 to NUM_RUNS) |
| `attempt` | Retry attempt within that run (1 to MAX_ATTEMPTS) |
| `response_from_llm` | The LLM's response text (e.g., "Agree") |
| `response_score` | Numeric score from the responses mapping |
| `llm_response_id` | Unique OpenAI response ID |

Example:
```csv
question_id,run_number,attempt,response_from_llm,response_score,llm_response_id
TT4G35A,1,1,Agree,3,resp_09742b27f3419024006973ca12171881939cc3c4d5617f9186
TT4G35A,2,1,Strongly agree,4,resp_0e3bf0f865bd6eeb006973ca203cc8819cade2c3b4614af3dd
```

### Response JSONs (`output/responses/*.json`)

Full OpenAI API response objects stored for audit/debugging purposes. Each file is named with the response ID.

## Retry Logic

If the LLM returns a response not in the valid options:
1. The response is still saved to `output/responses/`
2. A warning is printed
3. The statement is retried (up to MAX_ATTEMPTS)
4. Only successful responses are logged to CSV


