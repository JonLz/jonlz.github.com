# Start with a bare bones image that already has hugo installed
FROM klakegg/hugo:0.64.1-alpine

RUN cd hugo

# My theme is configured as a submodule so I need to pull it in. You may not
# need this if you have committed your theme directly to your repository.
RUN git submodule update --init --recursive

# Checkout the gh-pages branch as a working tree so that generated hugo files
# are ready to be committed in that branch.
RUN git worktree add -B gh-pages public origin/gh-pages

# Build the site
RUN hugo -t hugo-code -d ..

# Commit and push (NOTE: --allow-empty is here just in case I push a change that
# results in no change to the generated site, like updating a README or
# these build scripts).
RUN cd .. && \
    git add --all && \
    git commit -m "Site updated: `date +'%Y-%m-%d %H:%M:%S'`" --allow-empty && \
    git push origin master
