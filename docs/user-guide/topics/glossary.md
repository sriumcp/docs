---
template: main.html
---

# Iter8 Glossary

## A/B(/n) testing
A/B(/n) testing is the process of comparing two (or more) [versions](#version) of an [app](#app). Typically, the versions are compared in terms of [business metrics](#business-metrics), and the version that optimizes the business metrics is deemed as the [winner](#winner), and promoted as the [stable version](#stable-version) of the app.

## App
App is the subject of an Iter8 experiment, for instance, a microservice or an ML model. An app can have multiple versions.

## AutoX

## Builtin metrics

## Business metrics

## Canary testing

## Candidate version

## Custom metrics

## Distributed application

## Experiment
Experiment facilitates various release optimization use-cases through various pre-defined tasks. Typical tasks in an experiment include a) ensuring app versions are ready, b) generating load in performance testing experiments, c) performing traffic engineering A/B/n, canary, or traffic mirroring experiments, d) collecting business and/or performance metrics, e) assessing versions based on metrics, and f) promoting the winning version (for instance, through notifications).

## Iter8 CLI

## Iter8 SDK

## Iter8 SDK metrics

## Iter8 service

## Iter8 storage

## Metrics store

## Multi-loop experiment

## Performance metrics

## Performance testing

## Readiness
Kubernetes resources often have readiness conditions attached to their status. For instance, a Kubernetes deployment has its [Available](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) condition set to true if a minimum number of replicas of this deployment are up and running. Other custom resources like a [Knative service](https://knative.dev/docs/) or a [Kserve inference service](https://github.com/kserve/kserve) also define associated readiness conditions. Iter8 considers a Kubernetes resource to be ready if it exists, and the readiness condition defined by the resource (if any) is set to true. Iter8 considers a version to be ready if and only if every resource associated with the version is ready.

## Runner

## Service-level objective (SLO)

## Single-loop experiment

## Stable version

## Task

## User stickiness

## Version
Version of an app. A version can have one or more Kubernetes resources associated with it, and one or more metrics associated with it. Versions are numbered in Iter8 experiments.

## Version number
App versions are numbered in Iter8 experiments. In experiments involving multiple versions, it is typical for Version 1 to represent the stable version of the app, while Versions 2, 3, ... may represent candidate versions.

## Version promotion

## Winner

