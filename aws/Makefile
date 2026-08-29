# Global
AWS_REGION	:= us-east-1

# CloudFormations VARS
CF_STACKNAME	:=
CF_FILE			:=
CF_PROFILE		:= default
CF_TAGS			:=
CF_CHANGE_NAME	:=
CF_CHANGE_DESC	:=
ENV 			:=
# Space-separated ParameterKey=...,ParameterValue=... pairs (aws cli --parameters syntax).
# Defaults to the Env parameter for backward compatibility with per-environment
# stacks (e.g. tf-provisoner-role.yml). Override or clear (CF_PARAMS=) for
# templates with different/no parameters, e.g. github-oidc-provider.yml, which
# is a one-time, account-level stack with no Env parameter:
#   make cf-create CF_FILE=github-oidc-provider.yml CF_STACKNAME=... CF_PARAMS=
CF_PARAMS		:= ParameterKey=Env,ParameterValue=${ENV}
CF_PARAMS_FLAG	:= $(if $(strip ${CF_PARAMS}),--parameters ${CF_PARAMS},)

.PHONY: cf-create
cf-create:
	aws cloudformation create-stack \
	--stack-name ${CF_STACKNAME} \
	--template-body file://${CF_FILE} \
	${CF_PARAMS_FLAG} \
	--tags '${CF_TAGS}' \
	--capabilities "CAPABILITY_NAMED_IAM" \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-update
cf-update:
	aws cloudformation update-stack \
	--stack-name ${CF_STACKNAME} \
	--template-body file://${CF_FILE} \
	${CF_PARAMS_FLAG} \
	--capabilities "CAPABILITY_NAMED_IAM" \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-rm
cf-rm:
	aws cloudformation delete-stack \
	--stack-name ${CF_STACKNAME} \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-export
cf-export:
	aws cloudformation list-exports \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-create-change
cf-create-change:
	aws cloudformation create-change-set \
	--stack-name ${CF_STACKNAME} \
	--change-set-name ${CF_CHANGE_NAME} \
	--description "${CF_CHANGE_DESC}" \
	--template-body file://${CF_FILE} \
	--no-use-previous-template \
	--capabilities "CAPABILITY_NAMED_IAM" \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-describe-change
cf-describe-change:
	aws cloudformation describe-change-set \
	--change-set-name ${CF_CHANGE_NAME} \
	--stack-name ${CF_STACKNAME} \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-list-change
cf-list-change:
	aws cloudformation list-change-sets \
	--stack-name ${CF_STACKNAME} \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-apply-change
cf-apply-change:
	aws cloudformation execute-change-set \
	--change-set-name ${CF_CHANGE_NAME} \
	--stack-name ${CF_STACKNAME} \
	--no-disable-rollback \
	--color on \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}

.PHONY: cf-rm-change
cf-rm-change:
	aws cloudformation delete-change-set \
	--change-set-name ${CF_CHANGE_NAME} \
	--stack-name ${CF_STACKNAME} \
	--color on \
	--region ${AWS_REGION} \
	--output json \
	--profile ${CF_PROFILE}
