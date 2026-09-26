# Serialization and Saving

## SynalinksSaveable

**Purpose**: Marker base class for every serializable framework class.

**Single method**: `_obj_type()` (abstract; raises if not implemented). Used as a typecheck during deserialization.

## object_registration

**Purpose**: Register custom classes for deserialization without needing them passed at every call site.

**`register_synalinks_serializable(package="Custom", name=None)`** — Decorator:
```python
@register_synalinks_serializable(package="MyPackage", name="MyClass")
class MyClass:
    def get_config(self):
        return {...}
```

Registers in both `GLOBAL_CUSTOM_OBJECTS` (name→object) and `GLOBAL_CUSTOM_NAMES` (object→name).

**`CustomObjectScope` / `custom_object_scope`** — Context manager temporarily installing a `{name: obj}` dict into active scope (for per-call registration).

**`get_custom_objects()`** — Live reference to `GLOBAL_CUSTOM_OBJECTS` dict.

**`get_registered_name(obj)`** / **`get_registered_object(name, custom_objects=, module_objects=)`** — Lookup in either direction (checked in order: active scope, globals, caller-supplied, module_objects).

## serialization_lib

**Purpose**: Serialize/deserialize Synalinks objects (mirrors `keras.saving.serialization_lib`).

**`serialize_synalinks_object(obj)`** — Central dispatcher:
- Pass-through: `None`, plain types (`str/int/float/bool`)
- Recurse: `list/tuple/dict`
- Special: `bytes`, `slice`, `Ellipsis`, `SymbolicDataModel`, `JsonDataModel`, `lambda` (warns, dumps via `python_utils.func_dump`)
- Others: Build from `obj.get_config()`, wrap with `{module, class_name, config, registered_name}`

**`deserialize_synalinks_object(config, custom_objects=, safe_mode=True, ...)`** — Inverse operation, consulting `_retrieve_class_or_fn` → `get_registered_object`.

**`SafeModeScope`** — Gates lambda deserialization (unsafe: arbitrary code execution).

**`ObjectSharingScope`** — Detects same Python object referenced multiple times, serializes once with `shared_object_id`, reconstructs shared reference on load.

## Round-Trip Machinery

Every serializable class uses this pattern:
```python
@register_synalinks_serializable()
class MyClass:
    def get_config(self):
        return {"field1": value1, ...}
    
    @classmethod
    def from_config(cls, config):
        return cls(**config)
```

Used by `Program.save()`, `Program.to_json()`, `KnowledgeBase.get_config()`, `Sandbox.get_config()`, etc.

## Related pages
- [Data Model Hierarchy](../core/data-model-hierarchy.md) — DataModel serialization
- [Module Call Lifecycle](../core/module-and-call-lifecycle.md) — Module serialization
- [Trainer](../training/trainer.md) — Program save/load
