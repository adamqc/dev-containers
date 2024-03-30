# Build stage with Spack pre-installed and ready to be used
FROM spack/ubuntu-noble:develop AS builder


# What we want to install and how we want to install it
# is specified in a manifest file (spack.yaml)
RUN mkdir -p /opt/spack-environment && \
set -o noclobber \
&&  (echo spack: \
&&   echo '  specs:' \
&&   echo '  - cmake' \
&&   echo '  - dealii@master+complex+mpi+petsc+p4est+python+trilinos~hdf5~gmsh~sundials~oce~cgal~assimp~symengine~examples~ginkgo~threads~vtk~opencascade' \
&&   echo '  - eigen' \
&&   echo '  - python@:3.10.99' \
&&   echo '  - mumps+mpi+metis+parmetis' \
&&   echo '  - mpich@:4.1.99 datatype-engine=dataloop' \
&&   echo '  - petsc+mumps+superlu-dist+complex~hdf5' \
&&   echo '  - taskflow@master' \
&&   echo '  - tetgen' \
&&   echo '  - nanoflann' \
&&   echo '  - valgrind' \
&&   echo '  packages:' \
&&   echo '    all:' \
&&   echo '      target:' \
&&   echo '      - x86_64' \
&&   echo '      providers:' \
&&   echo '        mpi: [mpich]' \
&&   echo '  concretizer:' \
&&   echo '    unify: true' \
&&   echo '  config:' \
&&   echo '    install_tree: /opt/software' \
&&   echo '  view: /opt/views/view') > /opt/spack-environment/spack.yaml

# Install the software, remove unnecessary deps
RUN cd /opt/spack-environment && spack env activate . && spack install --fail-fast && spack gc -y

# Modifications to the environment that are necessary to run
RUN cd /opt/spack-environment && \
    spack env activate --sh -d . > activate.sh


# Bare OS image to run the installed executables
FROM ubuntu:24.04

COPY --from=builder /opt/spack-environment /opt/spack-environment
COPY --from=builder /opt/software /opt/software

# paths.view is a symlink, so copy the parent to avoid dereferencing and duplicating it
COPY --from=builder /opt/views /opt/views

RUN { \
      echo '#!/bin/sh' \
      && echo '.' /opt/spack-environment/activate.sh \
      && echo 'exec "$@"'; \
    } > /entrypoint.sh \
&& chmod a+x /entrypoint.sh \
&& ln -s /opt/views/view /opt/view


RUN apt-get -yqq update && apt-get -yqq upgrade \
 && apt-get -yqq install build-essential clangd clang-format curl gfortran git libc6-dbg vim \
 && rm -rf /var/lib/apt/lists/*

RUN sed -i 's|/opt/spack/lib/spack/env/gcc/||g' /opt/view/lib/cmake/deal.II/deal.IIConfig.cmake
RUN . /opt/spack-environment/activate.sh && python -m ensurepip && python -m pip install autopep8 scipy numpy matplotlib pandas -i https://pypi.org/simple && rm -rf /root/.cache
RUN sh -c "$(curl -L https://raw.githubusercontent.com/deluan/zsh-in-docker/master/zsh-in-docker.sh)" -- -t robbyrussell -p vi-mode

ENTRYPOINT [ "/entrypoint.sh" ]
CMD [ "/bin/bash" ]

