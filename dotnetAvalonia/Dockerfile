# ARGUMENTS --------------------------------------------------------------------
##
# Board architecture
##
ARG IMAGE_ARCH=

##
# Base container version
##
ARG BASE_VERSION=3-8.0

##
# Directory of the application inside container
##
ARG APP_ROOT=

##
# Board GPU vendor prefix
##
ARG GPU=

# ARGUMENTS --------------------------------------------------------------------

# BUILD ------------------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build

ARG IMAGE_ARCH
ARG APP_ROOT

COPY . ${APP_ROOT}
WORKDIR ${APP_ROOT}

# build
RUN dotnet restore __change__.csproj && \
    if [ "$IMAGE_ARCH" = "arm64" ] ; then \
        export ARCH=${IMAGE_ARCH} ; \
    elif [ "$IMAGE_ARCH" = "armhf" ] ; then \
        export ARCH="arm" ; \
    elif [ "$IMAGE_ARCH" = "amd64" ] ; then \
        export ARCH="x64" ; \
    fi && \
    dotnet publish __change__.csproj -c Release -r linux-${ARCH} --no-self-contained && \
    # this if is a edge case when the ARCH and IMAGE_ARCH are equal, like arm64
    if [ "./bin/Release/net8.0/linux-${ARCH}" != "./bin/Release/net8.0/linux-${IMAGE_ARCH}" ]; then \
        mv ./bin/Release/net8.0/linux-${ARCH} ./bin/Release/net8.0/linux-${IMAGE_ARCH}; \
    fi

# BUILD ------------------------------------------------------------------------


# DEPLOY -----------------------------------------------------------------------
FROM --platform=linux/${IMAGE_ARCH} \
    commontorizon/dotnet-x-gtk:${BASE_VERSION} AS deploy

ARG IMAGE_ARCH
ARG GPU
ARG APP_ROOT

# stick to bookworm on /etc/apt/sources.list.d
RUN sed -i 's/sid/bookworm/g' /etc/apt/sources.list.d/debian.sources

# for vivante GPU we need some "special" sauce
RUN apt-get -q -y update && \
        if [ "${GPU}" = "-vivante" ] || [ "${GPU}" = "-imx8" ]; then \
            apt-get -q -y install \
            imx-gpu-viv-wayland-dev \
        ; else \
            apt-get -q -y install \
            libgl1 \
        ; fi \
    && \
    apt-get clean && apt-get autoremove && \
    rm -rf /var/lib/apt/lists/*

RUN apt-get -y update && apt-get install -y --no-install-recommends \
    libice6 \
    libsm6 \
    # ADD YOUR PACKAGES HERE
# DO NOT REMOVE THIS LABEL: this is used for VS Code automation
    # __torizon_packages_prod_start__
    # __torizon_packages_prod_end__
# DO NOT REMOVE THIS LABEL: this is used for VS Code automation
	&& apt-get clean && apt-get autoremove && rm -rf /var/lib/apt/lists/*

# Copy the application compiled in the build step to the $APP_ROOT directory
# path inside the container, where $APP_ROOT is the torizon_app_root
# configuration defined in settings.json
COPY --from=build ${APP_ROOT}/bin/Release/net8.0/linux-${IMAGE_ARCH}/publish ${APP_ROOT}

# TODO: if you need to run as root user, you can remove this line
USER torizon

# "cd" (enter) into the APP_ROOT directory
WORKDIR ${APP_ROOT}

# Command executed in runtime when the container starts
CMD ["./__change__"]

# DEPLOY -----------------------------------------------------------------------
