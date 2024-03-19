# Configuring Docker to Push to and Pull from DO Registry

To interact with your registry using the docker command-line interface (CLI), you need to configure docker using the DigitalOcean command-line tool, doctl.

1. Install doctl CLI `brew install doctl`
2. Create an API key in DO.
3. Run `doctl registry login` and login with API key, This command adds credentials to docker so that pull and push commands to your DigitalOcean registry will be authenticated.

You can then use the docker tag command to tag your image with the fully qualified destination path, and docker push to upload it.

## Commands

**Note that you will need to build your images using linux/amd64 to allow them to run on digital ocean instances.**

`docker tag <my-image> registry.digitalocean.com/<my-registry>/<my-image>`
`docker push registry.digitalocean.com/<my-registry>/<my-image>`

### Example

`docker build --platform=linux/amd64 -t registry.digitalocean.com/tpm-containers-test/tpm-backend:0.0.05 .`

`docker push registry.digitalocean.com/tpm-containers-test/tpm-backend:0.0.05`
