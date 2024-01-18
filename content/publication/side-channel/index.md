---
title: "Masking Floating-Point Number Multiplication and Addition of Falcon"
authors:
- admin
- Jiun-Peng Chen
date: "2024-1-15"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2024-01-15T00:00:00Z"

# Publication type.
# Accepts a single type but formatted as a YAML list (for Hugo requirements).
# Enter a publication type from the CSL standard.
publication_types: ["article-journal"]

# Publication name and optional abbreviated publication name.
publication: "[IACR Transactions on Cryptographic Hardware and Embedded Systems (TCHES)](https://tches.iacr.org/)"
publication_short: ""

abstract: In this paper, we provide the first masking scheme for floating-point number multiplication and addition to defend against recent side-channel attacks on Falcon's pre-image vector computation. Our approach involves a masked nonzero check gadget that securely identifies whether a shared value is zero. This gadget can be utilized for various computations such as rounding the mantissa, computing the sticky bit, checking the equality of two values, and normalizing a number. To support the masked floating-point number addition, we also developed a masked shift and a masked normalization gadget. Our masking design provides both first- and higher-order mask protection, and we demonstrate the theoretical security by proving the (Strong)-Non-Interference properties in the probing model. To evaluate the performance of our approach, we implemented unmasked, first-order, and second-order algorithms on an Arm Cortex-M4 processor, providing cycle counts and the number of random bytes used. We also report the time for one complete signing process with our countermeasure on an Intel-Core CPU. In addition, we assessed the practical security of our approach by conducting the test vector leakage assessment (TVLA) to validate the effectiveness of our protection. Specifically, our TVLA experiment results for second-order masking passed the test in 100,000 measured traces.

# Summary. An optional shortened abstract.
summary: "In this paper, we provide the first masking scheme for floating-point number multiplication and addition to defend against recent side-channel attacks on Falcon's pre-image vector computation."

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
