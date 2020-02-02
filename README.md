# wercker_oke_demo

This demo showcases how you can use Wercker, Oracle Kubernetes Engine and OCI Registry for complete DevOps

## Demo Overview

This demo uses a demo springboot REST API to illustrate how you can configure Wercker to integrate with GitHub's Springboot repository, and define the CI/CD pipeline with the following steps:

- Build SpringBoot REST API
- Generate Docker Image
- Push Docker Image to OCIR
- Pull Docker Image from OCIR and Deploy to OKE

This steps have been tested on Oracle Cloud Infrastructure Gen 2 as of 3 Feb 2020. Should you encounter any problems, please feel free to feedback to Yang Yang (y.yeung@oracle.com).

## Pre-Requisite

Before you can proceed with the following demo, please ensure that you have the following accounts:

- OCI Gen 2 Account with Container Pipeline (Wercker), OKE, OCI Registry Access
- GitHub Account
- Wercker Account

In addition, you should also have gone through the guide in http://github.com/yyeung13/oke_ocir_demo to setup OCIR/OKE as well as the OCI CLI to manage the kubernetes cluster.

## Demo Details

## Wrapping Up
