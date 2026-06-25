# docs.grilo.gal -- MkDocs Documentation Site

## Introduction

This repository contains the source code for the **docs.grilo.gal**
documentation website.\
It is built using MkDocs, a fast and simple static site generator
focused on project documentation.

The site content is written in Markdown and configured through a single
`mkdocs.yml` file, allowing easy maintenance and deployment.

Repository: https://github.com/visincito/mkdocs

------------------------------------------------------------------------

## Table of Contents

-   Introduction
-   Project Structure
-   Configuration

------------------------------------------------------------------------

## Project Structure

    .
    ├── docs/
    │   ├── index.md           # Home
    │   ├── about.md           # About
    │   ├── blog/              # Blog plugin content
    │   ├── images/            # Static images
    │   ├── voip/              # VoIP (Asterisk, Kamailio, Verbio)
    │   ├── kubernetes/        # Kubernetes (k8s, Microk8s, Kind, K9s)
    │   ├── proxmox/           # Proxmox guides
    │   └── pnetlab/           # PNETLab guides
    ├── mkdocs.yml         # MkDocs configuration file
    ├── requirements.txt   # Python dependencies
    └── .gitignore         # Git ignored files

------------------------------------------------------------------------

## Configuration

All site configuration is handled through:

`mkdocs.yml`

Common configurable options:

-   `site_name`
-   `nav`
-   `theme`
-   `plugins`
-   `markdown_extensions`

------------------------------------------------------------------------

