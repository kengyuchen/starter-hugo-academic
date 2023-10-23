---
title: "Masking Floating-Point Number Multiplication and Addition of Falcon (Under Revision)"
authors:
- admin
- Jiun-Peng Chen
date: "2023-10-15"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2023-10-23T00:00:00Z"

# Publication type.
# Accepts a single type but formatted as a YAML list (for Hugo requirements).
# Enter a publication type from the CSL standard.
publication_types: ["article-journal"]

# Publication name and optional abbreviated publication name.
publication: "[IACR Transactions on Cryptographic Hardware and Embedded Systems (TCHES)](https://tches.iacr.org/)"
publication_short: ""

abstract: In this paper, we present the first masking scheme for Falcon’s floating-point number multiplication and addition, which are crucial for the pre-image vector computation. Our approach involves a masked nonzero check algorithm that securely identifies whether a shared value is zero. This algorithm can be utilized for various computations such as rounding the mantissa, computing the sticky bit, checking the equality of two values, and normalizing a number. To support the masked floating-point number addition, we also developed a masked shift and a masked normalization algorithm. Our masking design provides both first- and higher-order mask protection. We demonstrate the theoretical security of our algorithms by proving the (Strong)-Non-Interferece properties in the probing model. To evaluate the performance of our approach, we implement unmasked, first-order, and second-order algorithms on the ARM Cortex-M4 processor, providing cycle counts and the number of random bytes used. We also report the time for one complete signing process with our countermeasure on the Intel-Core CPU. In addition, we assess the practical security of our approach by conducting the test vector leakage assessment (TVLA) to validate effectiveness of our protection. Specifically, our TVLA experiment results for second-order masking pass the test in 100,000 measured traces.

# Summary. An optional shortened abstract.
summary: "In this paper, we present the first masking scheme for Falcon’s floating-point number multiplication and addition, which are crucial for the pre-image vector computation."

tags:
- Side-Channel Analysis

featured: true

links:
# - name: ''
  # url: ''
url_pdf: ''
url_code: ''
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/s9CC2SKySJM)'
  focal_point: ""
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""
---
