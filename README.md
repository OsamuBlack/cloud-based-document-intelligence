# End-to-End Document Analysis and Reporting using Intelligent Document Processing and Generative AI on Cloud

This repository contains the test results and related code for the research paper titled "End-to-End Document Analyzing and Reporting using Intelligent Document Processing and Generative AI using Cloud."

## Abstract
The research explores the digitization of documents using technologies such as Intelligent Document Processing (IDP) and Optical Character Recognition (OCR), converting unstructured and semi-structured data into structured, machine-readable formats. This structured data is then analyzed and reported using Generative AI, all implemented using cloud services.

## Introduction
Document automation workflow is crucial for efficient document management. This research paper presents an end-to-end solution leveraging cloud computing, OCR, IDP, and Generative AI to automate the process.

## Workflow Overview
The document automation workflow involves:
- Document Digitization
- Text Extraction
- Intelligent Document Processing
- Prompt-based Statistics
- Document Generation

## Empirical Design
The empirical evaluation assesses the accuracy and cost of OCR and Generative AI capabilities for various document types using cloud services.

## Research Questions
- RQ1: Accuracy of cloud-based pre-trained document intelligence models.
- RQ2: Effectiveness of cloud-based OCRs in extracting structured information.
- RQ3: Effort required to train a custom model.
- RQ4: Effectiveness of Generative AI in analyzing and summarizing documents.
- RQ5: Estimated costs of the solution.

## Empirical Results
The results section provides detailed findings for each research question, highlighting the performance and cost-effectiveness of the proposed solutions.

## Conclusion
The research demonstrates the potential of cloud technologies and Generative AI in transforming document management processes, offering significant operational efficiency and cost reduction.

## Repository Structure
- `azure/`: Root folder containing all the test data and results for azure document intelligence.
  - `pre-trained/`: Contains reciepts and layout folders which in turn contain the data related to these models of azure DI.
    - `layout/`: An input file named `sample.pdf` and its following outputs
      - **result.json**: Contains all the results of the file
      - **markdown-API3.1.md**: Markdown file generated using 3.1 API model of azure document intelligence.
      - **headings.txt**: Headings extracted from the document using custom code.
      - **result-image.png**: Shows the bounding boxes detected against the text.
     - `reciepts/`: Contains three folders, the samples for testing, the resulting images with bounding boxes, and the result json files.
  - `custom-model/resume`: Contains the training data used to train custom resume model, the samples to test the trained model and the results in both json and image format.
  - `generative-ai`: Contains the result of implementing an end-to-end document automation solution using azure document intelligence and azure open AI.

## Contact
For any inquiries or collaboration requests, please contact the authors via email provided in the paper.

