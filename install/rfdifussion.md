## rfdiffusion

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
-f ./assets/build/Dockerfile.rfdiffusion \
-t rfdiffusion:1.1.0 .
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
  rfdiffusion:1.1.0 \
  python3.9 /workdir/RFdiffusion/scripts/run_inference.py \
    inference.output_prefix=/app/test/rfdiff_design_with_target \
    inference.input_pdb=/workdir/RFdiffusion/examples/input_pdbs/1YCR.pdb \
    'contigmap.contigs=[A25-109/0 0-70/B17-29/0-70]' \
    contigmap.length=70-120 \
    'ppi.hotspot_res=[B19,B23,B27]' \
    inference.num_designs=10 \
    inference.ckpt_override_path=/app/checkpoints/Complex_base_ckpt.pt
```

Note 
* ` --cache /app/checkpoints` is important to save models locally and not have to redownload them every time you run app. 
* `--shm-size=64g` is important to allow model CPU memory sharing. Increase to reasonable limit your compute box allows. 


***Commdan breakdown***

1. **Input complex**  
   - `input_pdb=input_pdbs/1YCR.pdb`  
   - This is the crystal structure of p53 bound to Mdm2. We’ll treat Mdm2 (chain A, residues 25–109) as the fixed “target” and the p53 helix (chain B, residues 17–29) as the fixed “motif” we want to scaffold.

2. **Contig map**  
   - `'contigmap.contigs=[A25-109/0  0-70  /B17-29/0-70]'`  
     - **A25-109/0**: Keep Mdm2 exactly as is, no new design there.  
     - **0-70**: Before the p53 helix, allow the model to generate anywhere from 0 up to 70 new residues.  
     - **B17-29/0-70**: Keep the p53 helix motif fixed in the center.  
     - **0-70**: After the helix, again let the model fill in up to 70 residues.  

3. **Total length constraint**  
   - `contigmap.length=70-120`  
   - Forces the full designed chain (linker + motif + linker) to be between 70 and 120 amino acids long.

4. **Hotspots**
   - 'ppi.hotspot_res=[B19,B23,B27]'
   - By specifying ppi.hotspot_res=[B19,B23,B27], you’re saying “when you generate a new scaffold, make sure residues 19, 23, and 27 on chain B (the p53 motif) are tightly packed into the pocket.”
   - Those motif residues are the ones whose geometry and interactions you care about preserving or enhancing.

5. **How many designs?**  
   - `inference.num_designs=10`  
   - Generate ten different ways of wrapping that helix into a new protein scaffold.

6. **Which model checkpoint?**  
   - `inference.ckpt_override_path=…/Complex_base_ckpt.pt`  
   - Uses the version of RFDiffusion fine-tuned on protein–protein complexes, so it knows how to scaffold into a binding interface.

7. **Output**  
   - `inference.output_prefix=example_outputs/design_motifscaffolding_with_target`  
   - All ten designed backbone PDBs and helper files go into that folder.

---

***In Simple Terms***

- **You fix** the receptor (Mdm2) and the short helix motif (p53) in space.  
- **You tell** RFDiffusion “fill in before and after” to make a single continuous chain that holds that helix in the pocket.  
- **You constrain** how long the new chain can be.  
- **You ask** for multiple versions (10 designs) so you have variety.  
- **You use** a model already taught on real protein interfaces, so its scaffolds are more likely to bind tightly.

The result is ten brand-new protein backbones, each wrapping around the p53 helix in the Mdm2 pocket—ready for you to pick your favorites for further sequence design and experimental testing.