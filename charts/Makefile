CHARTS := $(shell find . -path '*/Chart.yaml' | tr '\n' ' ' | sed -E 's:\./|/Chart\.yaml::g')
DEP_CHARTS := $(shell find . -path '*/requirements.yaml' | tr '\n' ' ' |  sed -E 's:\./|/requirements\.yaml::g')

.PHONY: clean all package makepath copy index sync dependency-update
all: bump-changed package makepath copy index sync clean
prepare: bump-changed
dependency-update:
	helm init -c
	helm repo add lovii-charts https://lovii-charts.s3.amazonaws.com
	$(foreach chart,$(DEP_CHARTS),(helm dependency update --debug $(chart); echo $?) && ) :

package: dependency-update ; $(foreach chart,$(CHARTS),(helm package $(chart) --save=false) &&) :

makepath:
	@mkdir -p lovii-charts

copy:
	@aws s3 sync s3://lovii-charts lovii-charts
	@mv *.tgz lovii-charts/

index:
	@helm repo index ./lovii-charts --url https://lovii-charts.s3.amazonaws.com

sync:
	@aws s3 sync ./lovii-charts s3://lovii-charts

clean:
	@rm -rf lovii-charts

bump-changed: dependency-update
	./bump-changed.sh
