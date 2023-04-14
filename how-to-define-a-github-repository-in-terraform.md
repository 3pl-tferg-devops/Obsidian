```terraform
resource "github_repository" "my-app-public" {
	name      = "my-app-public"
	private   = false
	auto_init = true
}
```

[where-to-find-additional-configuration-for-defining-a-github-repository-in-terraform](https://registry.terraform.io/providers/integrations/github/latest/docs/resources/repository)



