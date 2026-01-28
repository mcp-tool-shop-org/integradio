# Contributing to Integradio

Thank you for your interest in contributing! This project provides vector-embedded Gradio components for semantic codebase navigation.

## Development Setup

```bash
git clone https://github.com/mcp-tool-shop/integradio.git
cd integradio
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e ".[dev,all]"
```

## How to Contribute

### Reporting Issues

If you find a bug or have a feature request:

1. Check existing [issues](https://github.com/mcp-tool-shop/integradio/issues)
2. If not found, create a new issue with:
   - Clear description of the problem or feature
   - Steps to reproduce (for bugs)
   - Expected vs. actual behavior
   - Your environment (Python version, Gradio version, OS)
   - Example code snippet if relevant

### Contributing Code

1. **Fork the repository** and create a branch from `main`
2. **Make your changes**
   - Follow the existing code style
   - Use type hints throughout
   - Add tests for new functionality
3. **Test your changes**
   ```bash
   pytest
   pytest --cov
   ```
4. **Commit your changes**
   - Use clear, descriptive commit messages
   - Reference issue numbers when applicable
5. **Submit a pull request**
   - Describe what your PR does and why
   - Link to related issues

## Project Structure

```
integradio/
├── integradio/
│   ├── __init__.py
│   ├── components.py      # Core Gradio components
│   ├── embeddings.py      # Embedding utilities
│   ├── search.py          # Semantic search logic
│   └── visualization.py   # Visualization helpers
├── behavior_modeler/      # Behavior modeling module
├── examples/              # Example applications
├── tests/                 # Test suite
└── pyproject.toml         # Project metadata
```

## Testing

Run the test suite:

```bash
pytest                      # All tests
pytest tests/test_components.py  # Specific test file
pytest -v                   # Verbose output
pytest --cov                # With coverage report
```

## Adding New Features

### Adding a New Gradio Component

1. Add component class in `integradio/components.py`
2. Implement embedding integration in `integradio/embeddings.py`
3. Add visualization helpers if needed in `integradio/visualization.py`
4. Create example in `examples/`
5. Add tests in `tests/`
6. Update README with component documentation

### Adding Embedding Support

1. Add embedding model integration in `integradio/embeddings.py`
2. Support configuration for model selection
3. Add tests with mock responses
4. Update documentation with model requirements

### Adding Search Capabilities

1. Extend search logic in `integradio/search.py`
2. Support HNSW and other vector indexes
3. Add performance benchmarks
4. Document search parameters

## Code Quality

We maintain code quality through:

### Type Checking
- Use type hints for all functions and methods
- Run type checkers before submitting PRs

### Testing
- Write tests for all new features
- Maintain test coverage above 80%
- Include integration tests for Gradio components

### Documentation
- Add docstrings to all public functions and classes
- Include type information in docstrings
- Provide usage examples in docstrings

## Running Examples

To test components locally:

```bash
cd examples
python example_semantic_search.py  # Specific example
```

Examples demonstrate:
- Semantic codebase search
- Vector visualization
- Embedding comparison
- Interactive UI components

## Design Principles

- **Gradio-first**: Components should integrate seamlessly with Gradio
- **Semantic search**: Prioritize semantic understanding over keyword matching
- **Flexible embeddings**: Support multiple embedding models (Ollama, OpenAI, etc.)
- **Type safety**: Full type coverage with hints
- **Performance**: Optimize for large codebases
- **User experience**: Intuitive, responsive UI components

## Common Tasks

### Using Semantic Search

```python
from integradio import SemanticSearchBox

search = SemanticSearchBox(
    embeddings_path="./embeddings.db",
    model="nomic-embed-text"
)
```

### Creating Custom Components

```python
from integradio import VectorVisualizer

viz = VectorVisualizer(
    dimensions=2,
    interactive=True
)
```

## Dependencies

### Core Dependencies
- **gradio**: UI framework
- **numpy**: Numerical operations
- **httpx**: HTTP client for API calls
- **pandas**: Data manipulation

### Optional Dependencies
- **hnswlib**: Fast approximate nearest neighbor search
- **fastapi**: API server support
- **uvicorn**: ASGI server

## Release Process

(For maintainers)

1. Update version in `pyproject.toml`
2. Update `CHANGELOG.md` with changes
3. Create git tag: `git tag v0.x.x`
4. Push tag: `git push origin v0.x.x`
5. GitHub Actions will build and publish to PyPI

## Questions?

Open an issue or start a discussion in the [MCP Tool Shop](https://github.com/mcp-tool-shop) organization.
