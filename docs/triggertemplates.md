# TriggerTemplates
A `TriggerTemplate` is a resource that can template resources.
`TriggerTemplate`s have optional parameters that can be substituted **anywhere** within the resource template.

<!-- FILE: examples/triggertemplates/triggertemplate.yaml -->
```YAML
apiVersion: tekton.dev/v1alpha1
kind: TriggerTemplate
metadata:
  name: pipeline-template
spec:
  params:
    - name: gitrevision
      description: The git revision
      default: master
    - name: gitrepositoryurl
      description: The git repository url
  resourcetemplates:
    - apiVersion: tekton.dev/v1alpha1
      kind: PipelineResource
      metadata:
        name: git-source-$(uid)
        labels:
          triggertemplated: "true"
          generatedBy: "triggers-example"
      spec:
        type: git
        params:
        - name: revision
          value: $(params.gitrevision)
        - name: url
          value: $(params.gitrepositoryurl)
    - apiVersion: tekton.dev/v1alpha1
      kind: PipelineRun
      metadata:
        generateName: simple-pipeline-run
        labels:
          triggertemplated: "true"
          generatedBy: "triggers-example"
      spec:
        pipelineRef:
            name: simple-pipeline
        resources:
        - name: git-source
          resourceRef:
            name: git-source-$(uid)
```

Similar to [Pipelines](https://github.com/tektoncd/pipeline/blob/master/docs/pipelines.md),`TriggerTemplate`s do not do any actual work, but instead act as the blueprint for what resources should be created.
Any parameters defined in the [`TriggerBinding`](triggerbindings.md) are passed into to the `TriggerTemplate` before resource creation.
If the namespace is omitted, it will be resolved to the `EventListener`'s namespace.

The `$(uid)` variable is implicitly available throughout a `TriggerTemplate`'s resource templates.
A random string value is assigned to `$(uid)` like the postfix generated by the Kubernetes `generateName` metadata field.
One instance where there is useful is when resources in a `TriggerTemplate` have internal references.

To enable support for any resource type, the resource templates are internally resolved as byte blobs.
As a result, validation on these resources is only done at event processing time (rather than during `TriggerTemplate` creation).