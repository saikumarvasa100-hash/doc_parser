Docling allows to enrich the conversion pipeline with additional steps which process specific document components,
e.g. code blocks, pictures, etc. The extra steps usually require extra models executions which may increase
the processing time consistently. For this reason most enrichment models are disabled by default.

The following table provides an overview of the default enrichment models available in Docling.

| Feature | Parameter | Processed item | Description |
| ------- | --------- | ---------------| ----------- |
| Code understanding | `do_code_enrichment` | `CodeItem` | See [docs below](#code-understanding). |
| Formula understanding | `do_formula_enrichment` | `TextItem` with label `FORMULA` | See [docs below](#formula-understanding). |
| Picture classification | `do_picture_classification` | `PictureItem` | See [docs below](#picture-classification). |
| Picture description | `do_picture_description` | `PictureItem` | See [docs below](#picture-description). |


## LLM/VLM Endpoint Configuration

### Overview

The enrichment preprocessors in the Intelligent Doc Parser can utilize Large Language Models (LLM) and Vision Language Models (VLM) for advanced document understanding tasks such as Table of Contents (TOC) extraction and metadata extraction. The configuration of these endpoints is crucial for proper operation.

**Important:** It is highly recommended to use the default preprocessor configurations provided by the system rather than manually tuning or customizing these settings. The default settings are optimized for most use cases and reduce the risk of configuration errors.

### Configuration Parameters

The following parameters control how the preprocessor connects to LLM/VLM services:

#### `toc_api_base_url` and `metadata_api_base_url` (Recommended)

- **Purpose**: Specifies the GenOS internal LLM/VLM serving endpoint URLs
- - **Usage**: This is the **primary and recommended method** for most deployments
  - - **Behavior**: When these URLs are provided, the system uses GenOS's internal logic to route requests to the appropriate models
    - - **Note**: When using these parameters, `toc_model` and `metadata_model` parameters are typically not required
     
      - Example configuration for GenOS deployment:
     
      - ```python
        pipeline_options.toc_api_base_url = "http://genos-llm-service:8080/v1"
        pipeline_options.metadata_api_base_url = "http://genos-llm-service:8080/v1"
        ```

        #### `toc_model` and `metadata_model` (Advanced Use Cases)

        - **Purpose**: Specifies model names or local paths for LLM/VLM execution
        - - **Usage**: Only use these parameters when NOT using GenOS serving (e.g., local models, OpenRouter, or other external services)
          - - **Behavior**: Accepts either a model identifier (e.g., `google/gemma-3-27b-it`) or a local filesystem path
            - - **Note**: These are legacy options maintained for backwards compatibility and special deployment scenarios
             
              - Example configuration for local model:
             
              - ```python
                pipeline_options.toc_model = "google/gemma-3-27b-it"
                pipeline_options.metadata_model = "/path/to/local/model"
                ```

                ### Recommended Usage Patterns

                #### Standard GenOS Deployment (Recommended)

                For most production deployments using GenOS infrastructure:

                ```python
                from docling.datamodel.pipeline_options import PdfPipelineOptions

                pipeline_options = PdfPipelineOptions()
                # Configure GenOS endpoints - this is the recommended approach
                pipeline_options.toc_api_base_url = "http://genos-llm-service:8080/v1"
                pipeline_options.metadata_api_base_url = "http://genos-llm-service:8080/v1"

                # No need to specify toc_model or metadata_model
                # GenOS will handle model selection internally
                ```

                #### Alternative Deployment (Advanced)

                For scenarios where GenOS serving is not available (e.g., local development, external API usage):

                ```python
                from docling.datamodel.pipeline_options import PdfPipelineOptions

                pipeline_options = PdfPipelineOptions()
                # Specify models directly (not using GenOS serving)
                pipeline_options.toc_model = "google/gemma-3-27b-it"
                pipeline_options.metadata_model = "google/gemma-3-27b-it"
                ```

                ### Best Practices

                1. **Use Default Configurations**: Always prefer using the provided default preprocessor rather than manual configuration
                2. 2. **Prefer API Base URLs**: When using GenOS, always configure `toc_api_base_url` and `metadata_api_base_url` instead of individual model parameters
                   3. 3. **Avoid Manual Tuning**: The field meanings and interactions can be complex; let the system handle configuration when possible
                      4. 4. **Future Enhancement**: In upcoming releases, these settings will be manageable through the GenOS UI and stored in the database, eliminating the need for code-level configuration
                        
                         5. ### Troubleshooting
                        
                         6. If you encounter configuration issues:
                        
                         7. - Ensure that `toc_api_base_url` and `metadata_api_base_url` are set if using GenOS infrastructure
                            - - Verify that the endpoints are accessible from your deployment environment
                              - - Avoid mixing configuration approaches (don't set both API URLs and model parameters unless you understand the precedence)
                                - - Consult with the GenOS team if you need custom configurations beyond the defaults
                                 
                                  - 

## Enrichments details

### Code understanding

The code understanding step allows to use advance parsing for code blocks found in the document.
This enrichment model also set the `code_language` property of the `CodeItem`.

Model specs: see the [`CodeFormula` model card](https://huggingface.co/ds4sd/CodeFormula).

Example command line:

```sh
docling --enrich-code FILE
```

Example code:

```py
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling.datamodel.pipeline_options import PdfPipelineOptions
from docling.datamodel.base_models import InputFormat

pipeline_options = PdfPipelineOptions()
pipeline_options.do_code_enrichment = True

converter = DocumentConverter(format_options={
    InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_options)
})

result = converter.convert("https://arxiv.org/pdf/2501.17887")
doc = result.document
```

### Formula understanding

The formula understanding step will analize the equation formulas in documents and extract their LaTeX representation.
The HTML export functions in the DoclingDocument will leverage the formula and visualize the result using the mathml html syntax.

Model specs: see the [`CodeFormula` model card](https://huggingface.co/ds4sd/CodeFormula).

Example command line:

```sh
docling --enrich-formula FILE
```

Example code:

```py
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling.datamodel.pipeline_options import PdfPipelineOptions
from docling.datamodel.base_models import InputFormat

pipeline_options = PdfPipelineOptions()
pipeline_options.do_formula_enrichment = True

converter = DocumentConverter(format_options={
    InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_options)
})

result = converter.convert("https://arxiv.org/pdf/2501.17887")
doc = result.document
```

### Picture classification

The picture classification step classifies the `PictureItem` elements in the document with the `DocumentFigureClassifier` model.
This model is specialized to understand the classes of pictures found in documents, e.g. different chart types, flow diagrams,
logos, signatures, etc.

Model specs: see the [`DocumentFigureClassifier` model card](https://huggingface.co/ds4sd/DocumentFigureClassifier).

Example command line:

```sh
docling --enrich-picture-classes FILE
```

Example code:

```py
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling.datamodel.pipeline_options import PdfPipelineOptions
from docling.datamodel.base_models import InputFormat

pipeline_options = PdfPipelineOptions()
pipeline_options.generate_picture_images = True
pipeline_options.images_scale = 2
pipeline_options.do_picture_classification = True

converter = DocumentConverter(format_options={
    InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_options)
})

result = converter.convert("https://arxiv.org/pdf/2501.17887")
doc = result.document
```


### Picture description

The picture description step allows to annotate a picture with a vision model. This is also known as a "captioning" task.
The Docling pipeline allows to load and run models completely locally as well as connecting to remote API which support the chat template.
Below follow a few examples on how to use some common vision model and remote services.


```py
from docling.document_converter import DocumentConverter, PdfFormatOption
from docling.datamodel.pipeline_options import PdfPipelineOptions
from docling.datamodel.base_models import InputFormat

pipeline_options = PdfPipelineOptions()
pipeline_options.do_picture_description = True

converter = DocumentConverter(format_options={
    InputFormat.PDF: PdfFormatOption(pipeline_options=pipeline_options)
})

result = converter.convert("https://arxiv.org/pdf/2501.17887")
doc = result.document

```

#### Granite Vision model

Model specs: see the [`ibm-granite/granite-vision-3.1-2b-preview` model card](https://huggingface.co/ibm-granite/granite-vision-3.1-2b-preview).

Usage in Docling:

```py
from docling.datamodel.pipeline_options import granite_picture_description

pipeline_options.picture_description_options = granite_picture_description
```

#### SmolVLM model

Model specs: see the [`HuggingFaceTB/SmolVLM-256M-Instruct` model card](https://huggingface.co/HuggingFaceTB/SmolVLM-256M-Instruct).

Usage in Docling:

```py
from docling.datamodel.pipeline_options import smolvlm_picture_description

pipeline_options.picture_description_options = smolvlm_picture_description
```

#### Other vision models

The option class `PictureDescriptionVlmOptions` allows to use any another model from the Hugging Face Hub.

```py
from docling.datamodel.pipeline_options import PictureDescriptionVlmOptions

pipeline_options.picture_description_options = PictureDescriptionVlmOptions(
    repo_id="",  # <-- add here the Hugging Face repo_id of your favorite VLM
    prompt="Describe the image in three sentences. Be consise and accurate.",
)
```

#### Remote vision model

The option class `PictureDescriptionApiOptions` allows to use models hosted on remote platforms, e.g.
on local endpoints served by [VLLM](https://docs.vllm.ai), [Ollama](https://ollama.com/) and others,
or cloud providers like [IBM watsonx.ai](https://www.ibm.com/products/watsonx-ai), etc.

_Note: in most cases this option will send your data to the remote service provider._

Usage in Docling:

```py
from docling.datamodel.pipeline_options import PictureDescriptionApiOptions

# Enable connections to remote services
pipeline_options.enable_remote_services=True  # <-- this is required!

# Example using a model running locally, e.g. via VLLM
# $ vllm serve MODEL_NAME
pipeline_options.picture_description_options = PictureDescriptionApiOptions(
    url="http://localhost:8000/v1/chat/completions",
    params=dict(
        model="MODEL NAME",
        seed=42,
        max_completion_tokens=200,
    ),
    prompt="Describe the image in three sentences. Be consise and accurate.",
    timeout=90,
)
```

End-to-end code snippets for cloud providers are available in the examples section:

- [IBM watsonx.ai](../examples/pictures_description_api.py)


## Develop new enrichment models

Beside looking at the implementation of all the models listed above, the Docling documentation has a few examples
dedicated to the implementation of enrichment models.

- [Develop picture enrichment](../examples/develop_picture_enrichment.py)
- [Develop formula enrichment](../examples/develop_formula_understanding.py)
