FROM ollama/ollama

RUN ollama serve & \
    sleep 5 && \
    ollama pull llama3 && \
    ollama pull nomic-embed-text