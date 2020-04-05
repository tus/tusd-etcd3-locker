# tusd-etcd3-locker

This repository contains an etcd3-backed locker for tusd for handling concurrent uploads.

To initialize a locker, a pre-existing connected etcd3 client must be present.

```
  client, err := clientv3.New(clientv3.Config{
      Endpoints:   []string{harness.Endpoint},
      DialTimeout: 5 * time.Second,
  })
```

Import the package:

```
import "github.com/tusd/tusd-etcd3-locker/pkg/etcd3locker"
```

For the most basic locker (e.g. non-shared etcd3 cluster / use default TTLs), a locker can be instantiated like the following:

```
  locker, err := etcd3locker.New(client)
  if err != nil {
      return nil, fmt.Errorf("Failed to create etcd locker: %v", err.Error())
  }
```

The locker will need to be included in composer that is used by tusd:

```
  composer := handler.NewStoreComposer()
  locker.UseIn(composer)
```

For a shared etcd3 cluster, you may want to modify the prefix that etcd3locker uses to ensure proper namespacing:

```
  locker, err := etcd3locker.NewWithPrefix(client, "my-prefix")
  if err != nil {
      return nil, fmt.Errorf("Failed to create etcd locker: %v", err.Error())
  }
```

For full control over all options, an etcd3.LockerOptions may be passed into etcd3.NewWithLockerOptions like the following example:

```
  ttl := 15 // seconds
  options := etcd3locker.NewLockerOptions(ttl, "my-prefix")
  locker, err := etcd3locker.NewWithLockerOptions(client, options)
  if err != nil {
      return nil, fmt.Errorf("Failed to create etcd locker: %v", err.Error())
  }
```

As of this time of writing, this locker has been tested on etcd versions 3.1/3.2/3.3.