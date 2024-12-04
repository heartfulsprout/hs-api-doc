API documentation for Heartful Sprout
Last updated: November 26, 2024



### Run
docker run --rm --name slate -p 4567:4567 -v $(pwd)/source:/srv/slate/source slatedocs/slate serve


### Build
docker run --rm --name slate -v $(pwd)/build:/srv/slate/build -v $(pwd)/source:/srv/slate/source slatedocs/slate build

Then, push it to Github

### Deploy
./deploy.sh --push-only