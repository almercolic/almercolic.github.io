---
layout: post
title:  "Terraform GKE node pool creation without downtime"
categories: gcp gke terraform node pool no downtime
render_with_liquid: false
---

The GKE Cluster can have one or more node pools that are used to deploy the Pods and Deployments. In order to provision the GKE cluster with terraform the  resource `google_container_cluster` is used. Nodepools are created with the resource `google_container_node_pool`.   
```
resource "google_container_node_pool" "nodepool" {
  project    = google_project.project.project_id
  name       = var.nodepool_name
  location   = var.nodepool_location
  cluster    = google_container_cluster.cluster.name
  node_count = var.nodepool_node_count

  node_config {
    preemptible     = true
    machine_type    = var.nodepool_machine_type
    service_account = google_service_account.default.email
  }
}
```

When we decide to apply a change in the configuration of the node pool, first the current node pool is being deleted and afterwards the new one is being created. The deletion of a node pool takes couple of minutes for a small number of nodes and increases with the node count. Since the nodes are drained our deployments are unsheduled and they suffer from a downtime until all the nodes are shut down and the new ones are ready to be used. 

Changing the order deletion -> creation to creation -> deletion this downtime is reduced to a minimum. As a prerequisite the node pool name needs to be unique, therefore we could add a random suffix to the name. The order is changed via the `create_before_destroy` lifecycle argument:
```
resource "google_container_node_pool" "nodepool" {
  project    = google_project.project.project_id
  name       = "${var.nodepool_name}-${random_integer.random.result}"
  location   = var.nodepool_location
  cluster    = google_container_cluster.cluster.name
  node_count = var.nodepool_node_count

  node_config {
    preemptible     = true
    machine_type    = var.nodepool_machine_type
    service_account = google_service_account.default.email
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

The last necessary step is to let the random integer be re-calculated each time any of the resource configurations are changed. This can be achieved with adding all the used inputs of the node pool resource in the keepers block to the random integer:
```
resource "random_integer" "random" {
  max = 999
  min = 100
  keepers = {
    name : var.nodepool_name,
    project : google_project.project.project_id,
    cluster : google_container_cluster.cluster.name,
    machine_type : var.nodepool_machine_type,
    location : var.nodepool_location,
    service_account : google_service_account.default.email,
    oauth_scopes : join(",", sort(local.oauth_scopes)),
  }
}
```

An example that demonstates the functionality can be found on at this [GitHub Repo](https://github.com/almercolic/terraform-gke-nodepool-update)