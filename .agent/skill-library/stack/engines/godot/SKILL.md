---
name: godot
description: "Expert Godot 4.x engine guide covering GDScript best practices (static typing, signals, exports, @onready), scene tree architecture, node composition patterns, physics (CharacterBody2D/3D, Area, RigidBody), navigation (NavigationAgent), networking (MultiplayerSynchronizer, RPCs), resource management, animation (AnimationPlayer, AnimationTree, Tweens), UI (Control nodes, themes), and export/deployment. Use when building games or interactive applications with Godot Engine."
version: 1.0.0
---

# Godot 4.x Expert Guide

> Use this skill when designing Godot projects, writing GDScript, structuring scene trees, implementing physics, networking, or UI. Targets Godot 4.x with GDScript (not C# or GDExtension).

## When to Use This Skill

- Building 2D or 3D games with Godot Engine
- Designing scene tree architecture and node hierarchies
- Implementing player controllers, AI, or physics interactions
- Setting up multiplayer/networking
- Creating UI systems with Control nodes
- Optimizing game performance

## When NOT to Use This Skill

- Unity or Unreal Engine projects
- Non-game Godot applications using C# → use C#-specific patterns
- Web-only applications → use a frontend framework

---

## 1. GDScript Best Practices (CRITICAL)

### Class Declaration Order

Follow the official style guide ordering:

```gdscript
class_name Player
extends CharacterBody2D

## Emitted when the player takes damage.
signal health_changed(new_health: int)
signal died

enum State { IDLE, RUN, JUMP, ATTACK }

const MAX_HEALTH: int = 100
const GRAVITY: float = 980.0

@export_group("Movement")
@export var move_speed: float = 200.0
@export var jump_force: float = -400.0

@export_group("Combat")
@export var attack_power: int = 25
@export var defense: int = 15

var health: int = MAX_HEALTH:
    set(value):
        health = clampi(value, 0, MAX_HEALTH)
        health_changed.emit(health)
        if health <= 0:
            died.emit()

var _state: State = State.IDLE
var _velocity_cache: Vector2 = Vector2.ZERO

@onready var animation_player: AnimationPlayer = $AnimationPlayer
@onready var sprite: Sprite2D = $Sprite2D
@onready var collision: CollisionShape2D = $CollisionShape2D


func _ready() -> void:
    pass

func _physics_process(delta: float) -> void:
    pass

func _unhandled_input(event: InputEvent) -> void:
    pass

# Public methods
func take_damage(amount: int) -> void:
    health -= max(0, amount - defense)

# Private methods
func _update_animation() -> void:
    pass
```

### Static Typing

**Always use static typing.** It catches bugs at parse time and enables editor autocompletion:

```gdscript
# ✅ Typed
var speed: float = 300.0
var items: Array[Item] = []
var target: Node2D = null
func get_damage() -> int:
    return attack_power

# ❌ Untyped
var speed = 300.0
var items = []
func get_damage():
    return attack_power
```

> Enable `Text Editor > Completion > Add Type Hints` in Editor Settings.

### Signals

Prefer signals over direct references for decoupling:

```gdscript
# ✅ Signal-based communication
signal coin_collected(value: int)

func _on_pickup_area_body_entered(body: Node2D) -> void:
    if body is Player:
        coin_collected.emit(coin_value)
        queue_free()

# Connect in parent scene
func _ready() -> void:
    coin.coin_collected.connect(_on_coin_collected)

func _on_coin_collected(value: int) -> void:
    score += value
```

### Export Variables

Use `@export` with type hints and grouping for editor usability:

```gdscript
@export_group("Stats")
@export var max_health: int = 100
@export_range(0.0, 1.0, 0.05) var crit_chance: float = 0.1
@export var character_class: StringName = &"Warrior"

@export_group("References")
@export var projectile_scene: PackedScene
@export_file("*.json") var config_path: String
@export_node_path("Area2D") var hitbox_path: NodePath
```

---

## 2. Scene Tree Architecture (CRITICAL)

### Composition Over Inheritance

Build game entities by composing nodes, not deep inheritance chains:

```
Player (CharacterBody2D)
├── Sprite2D
├── CollisionShape2D
├── AnimationPlayer
├── HitboxComponent (Area2D)
│   └── CollisionShape2D
├── HurtboxComponent (Area2D)
│   └── CollisionShape2D
├── HealthComponent (Node)
├── StateMachine (Node)
│   ├── IdleState
│   ├── RunState
│   └── AttackState
└── NavigationAgent2D
```

### Reusable Components

Extract shared behaviors into reusable scenes:

```gdscript
# health_component.gd
class_name HealthComponent
extends Node

signal health_changed(current: int, maximum: int)
signal died

@export var max_health: int = 100
var current_health: int

func _ready() -> void:
    current_health = max_health

func take_damage(amount: int) -> void:
    current_health = max(0, current_health - amount)
    health_changed.emit(current_health, max_health)
    if current_health <= 0:
        died.emit()

func heal(amount: int) -> void:
    current_health = min(max_health, current_health + amount)
    health_changed.emit(current_health, max_health)
```

### Autoloads (Singletons)

Use sparingly for truly global systems:

```gdscript
# game_manager.gd — registered as Autoload "GameManager"
extends Node

signal game_paused
signal game_resumed

var score: int = 0
var is_paused: bool = false

func pause_game() -> void:
    is_paused = true
    get_tree().paused = true
    game_paused.emit()

func resume_game() -> void:
    is_paused = false
    get_tree().paused = false
    game_resumed.emit()
```

> **Rule**: Only Autoload managers for global state (AudioManager, GameManager, SaveManager). Never Autoload game entities.

---

## 3. Physics & Movement

### CharacterBody2D/3D

```gdscript
extends CharacterBody2D

@export var speed: float = 300.0
@export var jump_force: float = -400.0
@export var gravity: float = 980.0

func _physics_process(delta: float) -> void:
    # Gravity
    if not is_on_floor():
        velocity.y += gravity * delta

    # Jump
    if Input.is_action_just_pressed("jump") and is_on_floor():
        velocity.y = jump_force

    # Horizontal movement
    var direction: float = Input.get_axis("move_left", "move_right")
    velocity.x = direction * speed

    move_and_slide()
```

### Area2D for Detection

```gdscript
# hitbox_component.gd
extends Area2D

@export var damage: int = 10

func _on_body_entered(body: Node2D) -> void:
    if body.has_method("take_damage"):
        body.take_damage(damage)
```

### Navigation

```gdscript
extends CharacterBody2D

@export var speed: float = 200.0
@onready var nav_agent: NavigationAgent2D = $NavigationAgent2D

func set_target(target_pos: Vector2) -> void:
    nav_agent.target_position = target_pos

func _physics_process(delta: float) -> void:
    if nav_agent.is_navigation_finished():
        return

    var next_pos: Vector2 = nav_agent.get_next_path_position()
    var direction: Vector2 = global_position.direction_to(next_pos)
    velocity = direction * speed
    move_and_slide()
```

> **Tip**: For 100+ AI agents, enable `NavigationAgent.avoidance_enabled` and time-slice pathfinding over multiple frames.

---

## 4. State Machines

```gdscript
# state_machine.gd
class_name StateMachine
extends Node

@export var initial_state: State
var current_state: State

func _ready() -> void:
    for child in get_children():
        if child is State:
            child.state_machine = self
    current_state = initial_state
    current_state.enter()

func _physics_process(delta: float) -> void:
    current_state.physics_update(delta)

func _unhandled_input(event: InputEvent) -> void:
    current_state.handle_input(event)

func transition_to(target_state: State) -> void:
    current_state.exit()
    current_state = target_state
    current_state.enter()
```

```gdscript
# state.gd
class_name State
extends Node

var state_machine: StateMachine

func enter() -> void: pass
func exit() -> void: pass
func handle_input(_event: InputEvent) -> void: pass
func physics_update(_delta: float) -> void: pass
```

---

## 5. Resources (Data-Driven Design)

```gdscript
# weapon_data.gd
class_name WeaponData
extends Resource

@export var name: StringName
@export var damage: int
@export var attack_speed: float
@export var projectile_scene: PackedScene
@export var icon: Texture2D
```

Create `.tres` files in the editor to define weapon variations without code changes:

```gdscript
# Usage in a weapon node
@export var weapon_data: WeaponData

func attack() -> void:
    var projectile = weapon_data.projectile_scene.instantiate()
    projectile.damage = weapon_data.damage
    get_tree().current_scene.add_child(projectile)
```

---

## 6. Networking (Multiplayer)

### High-Level Multiplayer

```gdscript
# lobby.gd
extends Node

var peer: ENetMultiplayerPeer = ENetMultiplayerPeer.new()

func host_game(port: int = 7000) -> void:
    peer.create_server(port)
    multiplayer.multiplayer_peer = peer
    multiplayer.peer_connected.connect(_on_peer_connected)

func join_game(address: String, port: int = 7000) -> void:
    peer.create_client(address, port)
    multiplayer.multiplayer_peer = peer
```

### MultiplayerSynchronizer

Sync only essential state — never position every frame:

```gdscript
# In scene: MultiplayerSynchronizer node
# Sync properties: health, mana, state
# Do NOT sync: position (use client-side prediction at 10-20Hz corrections)
```

### RPCs

```gdscript
@rpc("any_peer", "call_local", "reliable")
func deal_damage(target_id: int, amount: int) -> void:
    if multiplayer.is_server():
        var target = get_node_or_null(str(target_id))
        if target:
            target.take_damage(amount)
```

> **Rule**: Client sends *intent* ("I cast spell Q at direction V"), server validates and calculates results. Never trust client-computed damage.

---

## 7. Animation

### AnimationPlayer + AnimationTree

```gdscript
@onready var anim_tree: AnimationTree = $AnimationTree
@onready var state_machine: AnimationNodeStateMachinePlayback = anim_tree["parameters/playback"]

func _physics_process(delta: float) -> void:
    anim_tree["parameters/blend_position"] = velocity.normalized()

    if is_attacking:
        state_machine.travel("attack")
    elif velocity.length() > 0:
        state_machine.travel("run")
    else:
        state_machine.travel("idle")
```

### Tweens (Procedural Animation)

```gdscript
func flash_damage() -> void:
    var tween: Tween = create_tween()
    tween.tween_property(sprite, "modulate", Color.RED, 0.1)
    tween.tween_property(sprite, "modulate", Color.WHITE, 0.1)

func smooth_camera_shake(intensity: float) -> void:
    var tween: Tween = create_tween()
    tween.tween_property(camera, "offset", Vector2(randf_range(-intensity, intensity), randf_range(-intensity, intensity)), 0.05)
    tween.tween_property(camera, "offset", Vector2.ZERO, 0.05)
```

---

## 8. UI (Control Nodes)

```gdscript
# hud.gd
extends CanvasLayer

@onready var health_bar: ProgressBar = %HealthBar
@onready var score_label: Label = %ScoreLabel

func update_health(current: int, maximum: int) -> void:
    health_bar.max_value = maximum
    health_bar.value = current

func update_score(score: int) -> void:
    score_label.text = "Score: %d" % score
```

> Use `%UniqueNode` (unique name) syntax for UI references — more robust than `$Path/To/Node`.

---

## 9. Common Anti-Patterns

1. **Deep inheritance chains** — use composition (component nodes) instead of 5-level class hierarchies
2. **`get_node()` with long paths** — use `@onready`, `%UniqueName`, or signals for decoupling
3. **Processing when idle** — use `set_physics_process(false)` / `set_process(false)` for inactive entities
4. **`String` for IDs** — use `StringName` (`&"my_id"`) for identifiers and dictionary keys
5. **Untyped GDScript** — always use static typing for safety and performance
6. **Giant scripts** — split into component nodes (HealthComponent, StateMachine, etc.)
7. **Client-authoritative multiplayer** — server must validate all game-affecting actions
8. **Syncing everything every frame** — sync corrections at 10-20Hz, use prediction client-side
9. **Too many Autoloads** — only for truly global managers, not game logic

---

## References

- [Godot 4.x Documentation](https://docs.godotengine.org/en/stable/)
- [GDScript Style Guide](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- [GDScript Static Typing](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/static_typing.html)
- [Best Practices](https://docs.godotengine.org/en/stable/tutorials/best_practices/)
- [Networking](https://docs.godotengine.org/en/stable/tutorials/networking/)
- MOBA patterns adapted from [thedivergentai/gd-agentic-skills](https://github.com/thedivergentai/gd-agentic-skills) (MIT)
