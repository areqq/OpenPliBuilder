# OpenPliBuilder
OpenPli Docker builder

Create docker image:
```
git clone https://github.com/areqq/OpenPliBuilder
cd OpenPliBuilder
docker build -t areqq/openplibuilder:latest  Docker
```

Run docker container
```
mkdir build
docker run -it --rm -v ./build:/build areqq/openplibuilder:latest
```

local ./build == /build in docker

```
git clone https://github.com/OpenPLi/openpli-oe-core.git
cd openpli-oe-core
make
export MACHINE=vuuno4kse
make image
```

Build one package:

```
export MACHINE=vuuno4kse
cd openpli-oe-core/build
source env.source
bitbake -k git
```

# Links
https://wiki.openpli.org/Information_for_Developers#Create_your_own_build

https://forums.openpli.org/topic/45586-building-openpli-using-a-docker-image/

https://github.com/technic/e2xvfb


