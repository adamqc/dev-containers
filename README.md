# deal.II Docker images

This repository automatically builds the finite element library `deal.II` and `mfem`
every week using GitHub Actions. The build is done by using `spack`, and will result
in three Docker images:

- `adamqc/dealii-real:latest`: `deal.II` build with PETSc real numbers
- `adamqc/dealii-complex:latest`: `deal.II` build with PETSc complex numbers
- `adamqc/mfem:latest`: `mfem` build with PETSc real numbers
