# Start with a bare bones image that already has hugo installed
FROM klakegg/hugo:0.64.1-alpine

# Allow setting GH_ACTION_DEPLOY_KEY via the docker build command
ARG GH_ACTION_DEPLOY_KEY

# We need git to commit and openssh to push
RUN apk add git openssh
RUN git config --global user.email "jon.lazar@gmail.com" && \
    git config --global user.name "Jon Lazar"

# Use our build arg to store our private key. Run ssh-keyscan so we aren't
# prompted to trust the github.com host when we push our changes
RUN mkdir -p ~/.ssh/ && \
    echo "$GH_ACTION_DEPLOY_KEY" > ~/.ssh/id_rsa && \
    chmod 600 ~/.ssh/id_rsa && \
    ssh-keyscan github.com >> ~/.ssh/known_hosts

WORKDIR /site/
ADD . /site/

# By default, GH Actions will run with your repo checked out using the https
# protocol. This prevents us from being able to push using our stored key, so 
# we change it to git@
RUN git remote rm origin && \
    git remote add origin git@github.com:jonlz/jonlz.github.com && \
    git fetch

# My theme is configured as a submodule so I need to pull it in. You may not
# need this if you have committed your theme directly to your repository.
RUN git submodule update --init --recursive

# Checkout the gh-pages branch as a working tree so that generated hugo files
# are ready to be committed in that branch.
RUN git worktree add -B gh-pages public origin/gh-pages

# Build the site of course
RUN hugo

# Commit and push (NOTE: --allow-empty is here just in case I push a change that
# results in no change to the generated site, like updating a README or
# these build scripts).
RUN cd public && \
    git add --all && \
    git commit -m "Site updated: `date +'%Y-%m-%d %H:%M:%S'`" --allow-empty && \
    git push origin gh-pages:gh-pages