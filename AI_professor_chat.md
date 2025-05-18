The `AI_professor_chat.py` script defines the `AIProfessorChat` class, which serves as an intelligent chat assistant for interacting with academic papers. Here's a breakdown of its core logic:

1.  **Initialization (`__init__`)**: Sets up logging, paths, conversation history (limited size), paper context variables, and initializes an `LLMClient` for communication with a Large Language Model. It anticipates a `retriever` object (for RAG) to be provided later.
2.  **Paper Context (`set_paper_context`)**: Allows external components to inform the chat assistant about the currently active academic paper by providing its ID and associated data (metadata, summaries, etc.).
3.  **Query Processing (`process_query_stream`)**: This is the central method for handling user input.
    *   Manages conversation history (adds new queries, trims old ones).
    *   Calls `_make_decision` to determine the best strategy for answering the query using an LLM call. This decision includes the response `emotion`, the `function` (strategy: direct answer, page analysis, macro retrieval, RAG retrieval), and potentially an `optimized_query`.
    *   Executes the chosen strategy:
        *   `direct_answer`: No extra context needed.
        *   `page_content_analysis`: Uses currently visible text provided as input.
        *   `macro_retrieval`: Fetches high-level paper summaries (`_get_macro_context`).
        *   `rag_retrieval`: Uses the `retriever` object to find specific relevant passages (`_get_rag_context`), potentially returning UI scroll information.
    *   Prepares the final prompt for the main LLM call (`_prepare_final_messages`), including system instructions (character, task), conversation history, and the user query augmented with the chosen context and emotion guidance.
    *   Calls the LLM to get a response streamed sentence by sentence.
    *   Yields each sentence along with the decided emotion (and scroll info for the first sentence if applicable).
    *   Records the full AI response in the conversation history.
4.  **Context Retrieval (`_get_macro_context`, `_get_rag_context`)**: Helper methods to fetch either high-level paper summaries or specific passages using the RAG retriever.
5.  **Decision Logic (`_make_decision`)**: Uses a separate LLM call with a specific "router" prompt (including history, paper status) to determine the optimal response strategy (`function`), tone (`emotion`), and potentially refine the user's `query`. Includes validation and retry logic.
6.  **Utilities (`_read_file`, `_validate_decision`, `record_assistant_response`)**: Helper functions for file reading, decision validation, and explicitly recording responses.

In summary, `AIProfessorChat` intelligently routes user queries about academic papers. It decides whether to answer directly, use visible content, consult paper summaries, or perform detailed retrieval (RAG), then constructs a context-rich prompt for an LLM to generate a streamed, emotionally-toned response.