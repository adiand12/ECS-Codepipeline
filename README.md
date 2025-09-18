# ECS-Codepipeline
his repo demonstrates a minimal AWS CodePipeline flow for ECS services. CodeBuild produces both imagedefinitions.json (for ECS rolling updates) and appspec.yaml (for CodeDeploy Blue/Green/Canary), keeping deployment manifests in lockstep with the image tag and Task Definition revision.
