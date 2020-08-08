I needed to run sklearn code (which I did not write) in AWS Lambda,
because Lambda took care of scaling from 0 to 1000 workers,
which we didn't want to do using EC2 (with celery).

It was my idea to use Lambda, so I needed to make it work.

I got an error importing the libraries.

I found instructions online for how to run a docker image
that resembles the base Lambda (amazon linux),
install the native dependencies using yum,
install the python packages using pip,
copy the shared object files out of the container,
create a Lambda zip file that includes all the required .so files,
and then because you can't place the .so files under /usr/lib64/ in the Lambda environment,
dlopen() the .so files using their path in your dir before you import the python packages.

That worked, and has been working for years.

I documented it well and presented it to two other devs so the setup
has a bus factor greater than 1, and I haven't touched it since.

I guess there must be a better way to do it these days,
but I didn't need to find out.

[The recipe](https://github.com/ryansb/sklearn-build-lambda)
