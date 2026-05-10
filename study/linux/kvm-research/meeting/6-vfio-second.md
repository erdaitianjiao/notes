# vfio二次整理

## 上次遗留的问题
todo 
1. vfio_group的字段container_next
2. device的bar空间，PCI config space
3. iommu和domain的关系
4. gpu io-device的架构

### 1. contianer_next
```c
int vfio_container_attach_group(struct vfio_container *container,
				struct vfio_group *group)
{
    ...
	group->container = container;
	group->container_users = 1;
    ...
	list_add(&group->container_next, &container->group_list);
    ...
}
```