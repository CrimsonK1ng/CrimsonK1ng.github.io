# Terraform or Code as Infra    


## Terra stuff

I have used terraform and terragrunt a lot in the past. In a specific instance I needed to customize some behaviour so that I could do some weird infra stuff within a Lambda, and not use AWS' native infra as code options.

So basic setup was that the lambda needed to build infra on demand. I won't go into detail on everything but the code itself was pretty fun. Lambdas have a golang runtime which allows me to use Terraforms libraries to get things going. An example:

```go

import (
	grunt "github.com/gruntwork-io/terragrunt/cli"
	"github.com/gruntwork-io/terragrunt/options"
)

...
hclPath := filepath.Join(dir, workloadPath[workload])

	fleetOptions, err := options.NewTerragruntOptions(hclPath)
	if err != nil {
		log.WithError(err).Debug("error creating options")
		return err
	}
	// setup the command
	fleetOptions.TerraformCommand = action
	// add this by default
	fleetOptions.TerraformCliArgs = append(fleetOptions.TerraformCliArgs, action, "-no-color")
	if action == "apply" || action == "destroy" {
		fleetOptions.TerraformCliArgs = append(fleetOptions.TerraformCliArgs, "-auto-approve")
	}
	for _, k := range terraVars {
		if k == "" {
			continue
		}
		fleetOptions.TerraformCliArgs = append(fleetOptions.TerraformCliArgs, fmt.Sprintf("-var '%s'", k))
	}
	fleetOptions.Logger.Logger.SetFormatter(&terraFormatter{
		log.TextFormatter{
			DisableColors: true,
			FullTimestamp: true,
		},
	})
	fleetOptions.LogLevel = log.DebugLevel
	fleetOptions.NonInteractive = true
	fleetOptions.AutoInit = true
	fleetOptions.AutoRetry = true

    fleetOptions.Env = parseEnvironmentVariables(os.Environ())

	err = grunt.RunTerragrunt(fleetOptions)
	if err != nil {
		log.WithError(err).Error("error running terragrunt")
		return err
	}
...
```

Terragrunt has some basic options they expose to the user in their codebase. They don't really expect people to use it like this as they make frequent changes which is expected. But what I am doing here is rebuilding their command line options so I can invoke their internal terraform command similar to how terragrunt functions.


The one gotcha while working with lambdas is that the main directory is not writeable so you need to do everything in a temp workspace on the lambda. This can be achieved by creating a temp folder using golang, and copying the readable terraform files into that workspace. Then in order to execute it you simply have to adjust the hcl path to point to the new temp folder as the root location. 