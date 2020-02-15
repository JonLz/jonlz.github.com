# Start with a bare bones image that already has hugo installed
FROM klakegg/hugo:0.64.1-alpine

# Build the site
RUN hugo -t hugo-coder -d ..

# Commit and push (NOTE: --allow-empty is here just in case I push a change that
# results in no change to the generated site, like updating a README or
# these build scripts).
RUN cd .. && \
    git add --all && \
    git commit -m "Site updated: `date +'%Y-%m-%d %H:%M:%S'`" --allow-empty && \
    git push origin master
