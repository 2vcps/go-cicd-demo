FROM golang
LABEL maintainer "jon.owings@gmail.com"
EXPOSE 80
ENV GOPATH=/go
COPY ./code $GOPATH/src/gowebapp
WORKDIR $GOPATH/src/gowebapp/
RUN go get && go install
ENTRYPOINT [ "/go/bin/gowebapp" ]