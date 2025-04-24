## boltz1

Boltz-1 is the state-of-the-art open-source model to predict biomolecular structures containing combinations of proteins, RNA, DNA, and other molecules

* [Predicting](https://github.com/jwohlwend/boltz/blob/main/docs/prediction.md)
* [Technical Note](https://www.biorxiv.org/content/10.1101/2024.11.19.624167v2)
* [Development](https://github.com/jwohlwend/boltz)
* [Slack](https://boltz-community.slack.com/join/shared_invite/zt-2zj7e077b-D1R9S3JVOolhv_NaMELgjQ#/shared-invite/email)

### Prequisit

1. Docker
2. Git
3. NVIDIA drivers
4. Cloud SDK
    - [GCP](./gcp_setup.md)
    - [AWS](./aws_setup.md)

### Install 

#### Locally

Pull repository

```bash
git clone https://github.com/gabenavarro/PepContainerLab.git
cd PepContainerLab
```

Build docker image

```bash
docker build \
-f ./assets/build/Dockerfile.boltz1 \
-t boltz1:0.4.1 .
```

#### Cloud

TODO: COMPLETE 

### Run

#### Locally

```bash
docker run --rm -it \
  --gpus all \
  --shm-size=64g \
  -v "$(pwd):/app" \
  boltz1:0.4.1 \
  boltz predict /app/test/boltz1.fasta \
    --recycling_steps 10 \
    --diffusion_samples 25 \
    --accelerator gpu \
    --out_dir /app/test \
    --cache /app/checkpoints \
    --use_msa_server
```

Note 
* ` --cache /app/checkpoints` is important to save models locally and not have to redownload them every time you run app. 
* `--shm-size=32g` is important to allow model CPU memory sharing. Increase to reasonable limit your compute box allows. 