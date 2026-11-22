This repo builds local-SEO @ID stacks (S3 ID page, GCS wrapper, BatchGeo map). README.md is the playbook:
read it before building or changing a client. Build with `node build.mjs <slug>`; never change a client's
`imageDate` after its images are live. Only text goes through the AWS MCP connector; images are uploaded by hand.
