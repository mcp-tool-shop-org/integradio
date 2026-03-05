---
title: Semantic Wrappers
description: The core semantic() function and specialized wrappers for Chatbot, ImageEditor, plots, and more.
sidebar:
  order: 2
---

Integradio provides two levels of wrapping: the universal `semantic()` function for any Gradio component, and specialized wrappers that extract richer metadata from complex components.

## The core `semantic()` function

`semantic()` works with any Gradio component. Pass it a component and an intent string:

```python
from integradio import semantic
import gradio as gr

name_input = semantic(
    gr.Textbox(label="Full Name"),
    intent="user enters their full name",
    tags=["form", "required"],
)
```

**Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `component` | Any Gradio component | The component to wrap |
| `intent` | `str` | Natural language description of what this component does |
| `tags` | `list[str]` (optional) | Additional tags for filtering and categorization |

The intent string is embedded into a vector via Ollama. Tags are stored as metadata for filtering.

## Specialized wrappers

For components with complex behavior, specialized wrappers automatically generate rich semantic tags based on the component's configuration.

### `semantic_chatbot` — Chatbot

```python
from integradio import semantic_chatbot

chat = semantic_chatbot(
    gr.Chatbot(label="Assistant"),
    persona="coder",
    supports_streaming=True,
    supports_like=True,
)
# Auto-tags: ["io", "conversation", "ai", "streaming",
#             "persona-coder", "code-assistant", "programming"]
```

**Parameters:** `persona` (str), `supports_streaming` (bool), `supports_like` (bool)

### `semantic_image_editor` — ImageEditor

```python
from integradio import semantic_image_editor

editor = semantic_image_editor(
    gr.ImageEditor(label="Edit"),
    use_case="inpainting",
    supports_masks=True,
    tools=["brush", "eraser"],
)
# Auto-tags: ["input", "media", "editor", "visual",
#             "inpainting", "masking", "tool-brush", "tool-eraser"]
```

**Parameters:** `use_case` (str), `supports_masks` (bool), `tools` (list[str])

### `semantic_annotated_image` — AnnotatedImage

```python
from integradio import semantic_annotated_image

detections = semantic_annotated_image(
    gr.AnnotatedImage(label="Detections"),
    annotation_type="bbox",
    entity_types=["person", "car", "dog"],
)
# Auto-tags: ["output", "media", "annotation", "bbox",
#             "detection", "detects-person", "detects-car", "detects-dog"]
```

**Parameters:** `annotation_type` (str: "bbox" | "segmentation" | "keypoint"), `entity_types` (list[str])

### `semantic_highlighted_text` — HighlightedText

```python
from integradio import semantic_highlighted_text

entities = semantic_highlighted_text(
    gr.HighlightedText(label="Entities"),
    annotation_type="ner",
    entity_types=["PERSON", "ORG", "LOC"],
)
# Auto-tags: ["output", "text", "annotation", "nlp", "ner",
#             "person-entity", "organization-entity", "location-entity"]
```

**Parameters:** `annotation_type` (str: "ner" | "sentiment" | "classification"), `entity_types` (list[str])

### `semantic_multimodal` — MultimodalTextbox

```python
from integradio import semantic_multimodal

vlm_input = semantic_multimodal(
    gr.MultimodalTextbox(label="Ask about images"),
    use_case="image_analysis",
    accepts_images=True,
)
# Auto-tags: ["input", "text", "multimodal", "vision",
#             "image-input", "image_analysis", "vlm"]
```

**Parameters:** `use_case` (str), `accepts_images` (bool), `accepts_audio` (bool), `accepts_video` (bool)

### `semantic_plot` — LinePlot / BarPlot / ScatterPlot

```python
from integradio import semantic_plot

metrics_chart = semantic_plot(
    gr.LinePlot(x="date", y="value"),
    chart_type="line",
    data_domain="metrics",
    axes=["date", "value"],
)
# Auto-tags: ["output", "visualization", "chart-line",
#             "timeseries", "domain-metrics"]
```

**Parameters:** `chart_type` (str: "line" | "bar" | "scatter"), `data_domain` (str), `axes` (list[str])

### `semantic_model3d` — Model3D

```python
from integradio import semantic_model3d

viewer = semantic_model3d(
    gr.Model3D(label="3D Preview"),
    format="glb",
    use_case="product_preview",
)
# Auto-tags: ["output", "media", "3d", "glb", "product_preview"]
```

**Parameters:** `format` (str), `use_case` (str)

### `semantic_dataframe` — DataFrame

```python
from integradio import semantic_dataframe

table = semantic_dataframe(
    gr.Dataframe(label="Results"),
    editable=True,
    columns=["name", "score", "status"],
)
# Auto-tags: ["io", "data", "tabular", "editable",
#             "col-name", "col-score", "col-status"]
```

**Parameters:** `editable` (bool), `columns` (list[str])

### `semantic_file_explorer` — FileExplorer

```python
from integradio import semantic_file_explorer

files = semantic_file_explorer(
    gr.FileExplorer(label="Project Files"),
    root_dir="./workspace",
    glob_pattern="**/*.py",
)
# Auto-tags: ["io", "filesystem", "explorer", "python-files"]
```

**Parameters:** `root_dir` (str), `glob_pattern` (str)

## Auto-tags explained

Every specialized wrapper generates tags automatically based on:

1. **Direction** — `input`, `output`, or `io` (bidirectional)
2. **Domain** — `media`, `text`, `data`, `visualization`, `filesystem`, `conversation`
3. **Capabilities** — `streaming`, `masking`, `editable`, etc.
4. **Specifics** — entity types, tool names, column names, chart types

These tags are stored alongside the vector embedding and can be used for filtered search:

```python
# Search only among visualization components
results = demo.search("show me metrics", tags=["visualization"])
```
