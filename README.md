API documentation for Heartful Sprout
Last updated: November 26, 2024


### Build
docker run --rm --name slate -v $(pwd)/build:/srv/slate/build -v $(pwd)/source:/srv/slate/source slatedocs/slate build

### Run
docker run --rm --name slate -p 4567:4567 -v $(pwd)/source:/srv/slate/source slatedocs/slate serve

### Deploy
./deploy.sh