---
layout: post
title:  "Fast.ai PDLC - day 3"
date:   2023-10-05 18:16:10 -0700
categories: ML Learning fast.ai PDLC
---

# Summary
Primarirly we deployed a Gardio app in HuggingFace Space.

# Next steps
1. Export model
2. Deploy it to app

# Hugging face Spaces Git setup
1. Setup ssh key for git
2. Pull using git clone git@hf.co:spaces/USERNAME/REPONAME
and push should work.

# Covered
## Resizing
1. squish
2. randomResize: get different bits of image each time you run on same image
3. augumentation_transform: does some random transform to image

## Confusion matrix
Tracks errors in classification done by model.

## Image classifer cleaner
Train model for a specific category it's making most mistakes in