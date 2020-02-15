# Start with a bare bones image that already has hugo installed
FROM klakegg/hugo:0.64.1-alpine

# My theme is configured as a submodule so I need to pull it in. You may not
# need this if you have committed your theme directly to your repository.
RUN git submodule update --init --recursive

# Build the site
RUN hugo -t hugo-coder -d ..

# Commit and push (NOTE: --allow-empty is here just in case I push a change that
# results in no change to the generated site, like updating a README or
# these build scripts).
RUN cd .. && \
    git add --all && \
    git commit -m "Site updated: `date +'%Y-%m-%d %H:%M:%S'`" --allow-empty && \
    git push origin master
