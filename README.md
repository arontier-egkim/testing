```wdl
version 1.0

task molfinder_de_novo_1 {
  runtime {
    cpus: "${cpus}"
    docker: "registry.gitlab.com/arontier/workflows/genesis:2.7.4"
    docker_opts:
        " --volume ${input_path}:${input_path}:ro"
      + " --volume ${output_path}:${output_path}"
      + " --volume ${payload_path}:${payload_path}"
      + " --volume /seer/ad3/volumes/DB/MolFinder:/Arontier/Databases/MolFinder:ro"
      + " --volume /seer/ad3/volumes/models/SASCORE:/Arontier/AD3/models/SASCORE:ro"
  }
  input {
    String input_path
    String output_path
    String report_path = "${output_path}/final_bank"
    String payload_path = "${output_path}/../payload"
    Int cpus

    # project specific arguments
    Array[Float] model_weight_list
    Array[Float] weight_list
    Array[String] available_elements_type
    Array[String] filter_list
    Array[String] model_data_list
    Array[String] model_train_type
    Array[String] penalty_list
    Array[String] score_list
    Int local_min
    Int max_iteration
    Int nbank
    Int nseed
    Int repeat
    Array[String] model_data_path = prefix("${input_path}/", model_data_list)
    String available_elements_param = if length(available_elements_type) > 0 then "--available_elements_type" else ""
    String filter_param = if length(filter_list) > 0 then "--filter_list" else ""
    String model_data_param = if length(model_data_list) > 0 then "--model_data_list" else ""
    String model_train_param = if length(model_train_type) > 0 then "--model_train_type" else ""
    String model_weight_param = if length(model_weight_list) > 0 then "--model_weight_list" else ""
    String penalty_param = if length(penalty_list) > 0 then "--penalty_list" else ""
    String score_param = if length(score_list) > 0 then "--score_list" else ""
    String weight_param = if length(weight_list) > 0 then "--weight_list" else ""
  }
  command <<<
    # If the command fails and ${report_path} does not exist, create ${resport_path} and create "__FATAL__" file in the directory.
    trap 'exit_code=$?; [[ $exit_code -ne 0 ]] && (([[ ! -d "~{report_path}" ]] && mkdir -p "~{report_path}") || true) && touch "~{report_path}/__FATAL__"; exit $exit_code;' ERR

    # When the command ends (regardless of success or failure), symlink ${report_path} to ${payload_path}.
    trap 'ln -s "~{report_path}"/* "~{payload_path}";' EXIT

    # Define the environment variable "TMPDIR" to suppress "AF_UNIX path too long" error.
    # @see https://github.com/broadinstitute/cromwell/issues/3647
    export TMPDIR=/tmp

    ########## NEVER edit the above lines ##########

    # cd /molfinder/MolFinder-integrate
    gnss_molfinder \
      --nbank ~{nbank} \
      --nseed ~{nseed} \
      --max_iteration ~{max_iteration} \
      --repeat ~{repeat} \
      --ncpu ~{cpus} \
      ~{score_param} ~{sep=" " score_list} \
      ~{weight_param} ~{sep=" " weight_list} \
      ~{filter_param} ~{sep=" " filter_list} \
      ~{penalty_param} ~{sep=" " penalty_list} \
      ~{model_data_param} ~{sep=" " model_data_path} \
      ~{model_weight_param} ~{sep=" " model_weight_list} \
      ~{model_train_param} ~{sep=" " model_train_type} \
      ~{available_elements_param} ~{sep=" " available_elements_type} \
      --local_min ~{local_min} \
      --output ~{output_path}
  >>>
  output {
    File out = "${payload_path}"
  }
}

workflow molfinder_de_novo {
  input {
    String workflow_input
    String workflow_output
    Int cpus

    # project specific arguments
    Array[Float] model_weight
    Array[Float] weight
    Array[String] available_elements
    Array[String] filter
    Array[String] model_data
    Array[String] model_train_type
    Array[String] penalty
    Array[String] score
    Int local_min
    Int max_iteration
    Int nbank
    Int nseed
    Int repeat
  }
  call molfinder_de_novo_1 {
    input:
      input_path = workflow_input,
      output_path = workflow_output,
      cpus = cpus,

      # project specific arguments
      available_elements_type = available_elements,
      filter_list = filter,
      local_min = local_min,
      max_iteration = max_iteration,
      model_data_list = model_data,
      model_train_type = model_train_type,
      model_weight_list = model_weight,
      nbank = nbank,
      nseed = nseed,
      penalty_list = penalty,
      repeat = repeat,
      score_list = score,
      weight_list = weight
  }
}
```
