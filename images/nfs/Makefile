# Copyright 2018 The Rook Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

include ../image.mk

# ====================================================================================
# Image Build Options

NFS_IMAGE = $(BUILD_REGISTRY)/nfs-$(GOARCH)

NFS_BASE ?= fedora:28
DOCKERFILE ?= Dockerfile

ifeq ($(GOARCH),amd64)
	NFS_BASEIMAGE = $(NFS_BASE)
else ifeq ($(GOARCH),arm64)
	NFS_BASEIMAGE = arm64v8/$(NFS_BASE)
	DOCKERFILE = Dockerfile-arm64
endif

TEMP := $(shell mktemp -d)

# ====================================================================================
# Build Rook NFS

# since this is a leaf image we avoid leaving around a lot of dangling images
# by removing the last build of the final nfs image
OLD_IMAGE_ID := $(shell docker images -q $(NFS_IMAGE))
CURRENT_IMAGE_ID := $$(docker images -q $(NFS_IMAGE))
IMAGE_FILENAME := $(IMAGE_OUTPUT_DIR)/nfs.tar.gz

do.build:
	@echo === docker build $(NFS_IMAGE)
	if [ $(GOARCH) == arm64 ]; then \
		docker run --rm --privileged multiarch/qemu-user-static:register --reset; \
		sed -e '/FROM NFS_BASEIMAGE/r Dockerfile_header_arm64' Dockerfile > ${DOCKERFILE}; \
		cd $(TEMP) && wget https://github.com/multiarch/qemu-user-static/releases/download/v3.0.0/qemu-aarch64-static; \
		chmod a+x qemu-aarch64-static; \
	fi
	@cp $(DOCKERFILE) $(TEMP)
	@cp start.sh $(TEMP)
	@cp $(OUTPUT_DIR)/bin/linux_$(GOARCH)/rook $(TEMP)
	@cd $(TEMP) && $(SED_CMD) 's|NFS_BASEIMAGE|$(NFS_BASEIMAGE)|g' $(DOCKERFILE)
	@docker build $(BUILD_ARGS) \
		-t $(NFS_IMAGE) \
		-f $(TEMP)/$(DOCKERFILE) $(TEMP)
	@[ "$(OLD_IMAGE_ID)" != "$(CURRENT_IMAGE_ID)" ] && [ -n "$(OLD_IMAGE_ID)" ] && docker rmi $(OLD_IMAGE_ID) || true
	@if [ ! -e "$(IMAGE_FILENAME)" ] || [ "$(OLD_IMAGE_ID)" != "$(CURRENT_IMAGE_ID)" ] || [ -n "$(OLD_IMAGE_ID)" ]; then \
		echo === saving image $(NFS_IMAGE); \
		mkdir -p $(IMAGE_OUTPUT_DIR); \
		docker save $(NFS_IMAGE) | gzip -c > $(IMAGE_FILENAME); \
	fi
	@rm -fr $(TEMP)
