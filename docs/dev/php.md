# PHP

### Foreach

```php
<?php foreach($contacts as $contact): ?>
  <div class="col-md-4 mb-3">
    <div class="card text-center">
      <div class="card-body">
        <h3 class="card-title text-capitalize"><?= $contact["name"]; ?></h3>
        <p class="m-2"><?= $contact["phone_number"]; ?></p>
        <a href="#" class="btn btn-secondary mb-2">Edit Contact</a>
        <a href="#" class="btn btn-danger mb-2">Delete Contact</a>
      </div>
    </div>
  </div>
<?php endforeach ?>
```
