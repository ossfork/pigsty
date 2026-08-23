#==============================================================#
# File      :   Makefile
# Desc      :   OpenTofu / Terraform admin shortcuts
# Ctime     :   2024-12-24
# Mtime     :   2026-08-24
# Path      :   Makefile
# Author    :   Ruohang Feng (rh@vonng.com)
# License   :   Apache-2.0 @ https://pigsty.io/docs/about/license/
#==============================================================#


IAC_CLI ?= tofu


###############################################################
#                         IaC Commands                         #
###############################################################
check-cli:
	@command -v "$(IAC_CLI)" >/dev/null || { echo "$(IAC_CLI) is not installed" >&2; exit 1; }

init: check-cli
	$(IAC_CLI) init
validate: check-cli
	$(IAC_CLI) validate
plan: check-cli
	$(IAC_CLI) plan

u: up
up: check-cli
	$(IAC_CLI) apply
	sleep 15
	IAC_CLI="$(IAC_CLI)" ./ssh
up-auto: check-cli
	$(IAC_CLI) apply -auto-approve
	sleep 15
	IAC_CLI="$(IAC_CLI)" ./ssh
apply: check-cli
	$(IAC_CLI) apply
apply-auto: check-cli
	$(IAC_CLI) apply -auto-approve

d: destroy
destroy: check-cli
	$(IAC_CLI) destroy
destroy-auto: check-cli
	$(IAC_CLI) destroy -auto-approve
out: check-cli
	$(IAC_CLI) output
ssh: check-cli
	IAC_CLI="$(IAC_CLI)" ./ssh
r: reset
reset:
	git restore terraform.tf


###############################################################
#                         Inventory                           #
###############################################################
.PHONY: check-cli init validate plan u up up-auto apply apply-auto d destroy destroy-auto out ssh r reset
