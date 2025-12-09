# Slash
## Unreal Engine 5 C++ The Ultimate Game Developer Course
Hack and slash RPG game.

---

## Section 1 - Getting Started

### Hotkeys

- **Show FPS**: `CTRL + SHIFT + H`
- **Set Camera Bookmark**: `CTRL + (number)`
- **Show Camera Bookmark**: `(number)`
- **Gameview**: `G`
- **Immersive Mode**: `F11`

### Viewport

- **Show Collision**: Show => Collision
- **High Res. Screenshot**: Hamburger => Take High Resolution Screenshot

---

## Section 5 - The Actor Class

### Pointers

- Needs to be null-checked (e.g., `GEngine`, `GetWorld()`)
- Uses the arrow operator (`GEngine->AddOnScreenDebugMessage(...)`)

Example:
```cpp
if (GEngine)
{
    GEngine->AddOnScreenDebugMessage();
}
```

### Debug Messages

- `__VA_ARGS__`: number of arguments (e.g., `UE_LOG()`)
- `UE_LOG(LogTemp, Warning, TEXT("DeltaTime: %f"), DeltaTime);`

#### String Formatting
- Needs `*Name` for strings
```cpp
FString Message = FString::Printf(TEXT("Item Name: %s"), *Name);
```
- `%f` converts to float
- `%s` converts to string
- `%d` converts to int32

### Macros

#### Define variable & function

Include here instead of the inherited cpp file:
```cpp
#include "DrawDebugHelpers.h"
```

#### Variable
```cpp
#define THIRTY 30
```

#### Define with multiple lines
```cpp
#define DRAW_SPHERE(Location) \
if (GetWorld()) DrawDebugSphere(GetWorld(), Location, ...);

#define DRAW_VECTOR(StartLocation, EndLocation) \
if (GetWorld()) \
{ \
    DrawDebugLine(GetWorld(), StartLocation, EndLocation, FColor::Emerald, true, -1.f, 0, 1.f); \
    DrawDebugPoint(GetWorld(), StartLocation, 15.f, FColor::Emerald, true); \
}
```

---

## Section 6 - Moving Objects with Code

### Update movement rate with frame rate

```cpp
// Movement rate in units of cm/s
float MovementRate = 50.f;

// MovementRate * DeltaTime => (cm/s) * (s/frame) = (cm/frame)
AddActorWorldOffset(FVector(MovementRate * DeltaTime, 0.f, 0.f));
```

### Initialize value

Header file:
```cpp
float Amplitude = 0.25f;
```

### Expose properties and functions to blueprints

#### UPROPERTY

- **EditDefaultsOnly**: Changes all components; only in blueprint, not in the details panel
- **EditInstanceOnly**: Changes instance component only; only in the details panel
- **EditAnywhere**: Both default and individual panels
- **VisibleDefaultsOnly**: Read-only in blueprint
- **VisibleInstanceOnly**: Read-only in details panel; can be seen in runtime
- **VisibleAnywhere**: Read-only in both default and individual panels
- **BlueprintReadOnly**: Cannot be used on private variables (protected at least); exposed as a getter in blueprint
- **BlueprintReadWrite**: Exposed as both getter/setter in blueprint

#### Category

Sets the category tab in blueprint panel:
```cpp
Category = "Sine Parameters"
```

#### Meta specifiers

Allows private variables to be exposed in blueprint:
```cpp
meta = (AllowPrivateAccess = "true")
```

#### UFUNCTION

- **BlueprintCallable**: Makes callable in blueprint
- **BlueprintPure**: A return node in blueprint

### Template functions

Header file:
```cpp
template<typename T>
inline T AItem::Avg(T First, T Second)
{
    return (First + Second) / 2;
}
```

cpp file:
```cpp
int32 AvgInt = Avg<int32>(1, 3);
```

### Components

- `GetActorLocation()` is the Root Component's location
- Relative location is the location relative to the root

---

## Section 7 - The Pawn Class

### Forward Declaration

Forward declare above class:
```cpp
class UCapsuleComponent;

UCLASS()
class SLASH_API ABird : public APawn
{
    GENERATED_BODY()
}
```

### Hotkeys

- **Cursor in runtime**: `Shift + F1`
- **Show collision in runtime**: `§ + ShowCollision`
- **Show enhanced input in runtime**: `§ + showdebug enhancedinput`

### Enhanced Input

- **Input Mapping Context**: Sets to specific player (instead of project settings/inputs); a set of control mappings for different modes (e.g., Normal Character controls vs Character in Vehicle controls)

#### IA_Look

- Axis2D (`Vector2D`)

---

## Section 8 - The Character Class

### Character Movement

- **Orient rotation to movement**: Character faces the way of movement
- **Rotation rate**: How fast the character rotates with movement

### Cache

- **Binaries, Intermediate & Saved**: Can be deleted when rebuilding
- **Generate Visual Studio file**: Double-click `name.uproject` - Rebuild

### Hair

- Add `HairStrandsCore` and `Niagara` to `build.cs`

#### Example code for setting hair:
```cpp
Hair = CreateDefaultSubobject<UGroomComponent>(TEXT("Hair"));
Hair->SetupAttachment(GetMesh());
Hair->AttachmentName = FString("head");
```

---

## Section 9 - Animation Blueprint

### Animation Blueprint

#### State Machine

- **Idle to Run**: Transition rules between animation states
- **Idle node**: Sets the animation
- **Arrow from Idle to Run**: Sets the condition for transition

#### Cached Poses

- Save states for performance optimization

### C++ Variables to Blueprint

- **Show C++ Variables in Blueprint**: Cogwheel => Show Inherited Variables

### Inverse Kinematics

- **Sphere Trace**: Used for foot placement
- **Z-Offset Right Foot vs Left Foot**: Determines foot placement height
- **Interpolate**: Smoothly transitions towards a target position
- **Control Rig**: Uses IK-bones for advanced skeletal control
- **Skeleton Mesh**: Uses virtual bones to replace IK-bones

---

## Section 10 - Collision and Overlaps

### Collision Presets
- **Collision Enabled**:
  - No Collision
  - Query Only: Raycasts, sweeps, and overlaps
  - Physics Only: Physics collision, no queries
- **Object Type**:
  - Set object type to object: How other types would respond to it (ignore, overlap, and block)

### BP
- **Collision component**:
  - Hidden in game
- **On Component Begin Overlap**:
  - Component-specific event

### Delegates
#### Spider Queen
- Summon minions
- Screamer => Scream(float Volume)
  - Bind Summon minions to Scream
- Charger => Charge(float Speed)
  - Bind Summon minions to Charger
- `SummonMinions.Broadcast(50.f)` => `Scream(50.f);`

### Keyboard shortcut
- From header file to cpp: `CTRL + K + O`

### Collisions C++
#### Overlap Code in cpp file:
```cpp
void AItem::BeginPlay()
{
    Super::BeginPlay();
    
    // Binds delegates
    SphereComponent->OnComponentBeginOverlap.AddDynamic(this, &AItem::OnSphereOverlap);
    SphereComponent->OnComponentEndOverlap.AddDynamic(this, &AItem::OnSphereEndOverlap);
}
```

#### Overlap Code in header file:
```cpp
UFUNCTION()
void OnSphereOverlap(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
```

#### Delegate can be found in `PrimitiveComponent.h` with code:
```cpp
DECLARE_DYNAMIC_MULTICAST_SPARSE_DELEGATE_SixParams(FComponentBeginOverlapSignature, UPrimitiveComponent, OnComponentBeginOverlap, UPrimitiveComponent*, OverlappedComponent, AActor*, OtherActor, UPrimitiveComponent*, OtherComp, int32, OtherBodyIndex, bool, bFromSweep, const FHitResult&, SweepResult);
```

If the component does not trigger overlap, restart UE5.

---

## Section 11 - The Weapon Class

### General Notes
- Overrides cannot have the `UFUNCTION` macro.
- Add socket to skeletal mesh bone.

### Mixamo
- Assets for skeleton meshes and animations.

### Retargeting
- Maps animation for a skeletal mesh to another.

### IK_Rig
- **Chains**: For example, Spine to Spine2.
- Make corresponding chains for both rigs (e.g., Echo and X_Bot).

### IK_Retargeter
- Takes the two meshes and edits the bones.

### Picking up a Weapon
- **Blueprint Logic**:
  - `BP_Weapon` → On Component Overlap → Cast to `BP_SlashCharacter` → Attach `Item Mesh` (weapon) to component → Target: Get Mesh from `SlashCharacter` → Parent: `SocketName RightHandSocket`.

- **C++ Code**:
```cpp
Super::OnSphereOverlap(OverlappedComponent, OtherActor, OtherComp, OtherBodyIndex, bFromSweep, SweepResult);

ASlashCharacter* SlashCharacter = Cast<ASlashCharacter>(OtherActor);

if (SlashCharacter)
{
    FAttachmentTransformRules TransformRules(EAttachmentRule::SnapToTarget, true);
    ItemMesh->AttachToComponent(SlashCharacter->GetMesh(), TransformRules, FName("RightHandSocket"));
}
```

### Scoped Enum Example
#### Enum Declaration:
```cpp
UENUM(BlueprintType)
enum class ECharacterState : uint8
{
    ECS_Unequipped UMETA(DisplayName = "Unequipped"),
    ECS_EquippedOneHandedWeapon UMETA(DisplayName = "Equipped One-Handed Weapon"),
    ECS_EquippedTwoHandedWeapon UMETA(DisplayName = "Equipped Two-Handed Weapon")
};
```

### Getter and Setter
#### Setter:
```cpp
FORCEINLINE void SetOverlappingItem(AItem* Item) { OverlappingItem = Item; }
```

#### Getter:
```cpp
FORCEINLINE ECharacterState GetCharacterState() const { return CharacterState; }
```

### Animation Blueprints
- **Linked Animation Graph**.
- **Exposable Properties**: Send variables to main states.

## Section 12 - Attacking

### Animation Montage

- **Chaining Animations**: Chain animations such as different attacks.
- **Sections**: Attack 1, Attack 2.
- **Slots**: Slots are used in the Animation Blueprint to execute the montage.
- **Notifies**: Add a new notify to inform the Animation Blueprint when an attack ends.
  - Path: `Animation Blueprint -> Event Graph -> AnimNotify_AttackEnd`.

### Attack in C++
```cpp
void ASlashCharacter::Attack()
{
	if (CanAttack())
	{
		PlayAttackMontage();
		ActionState = EActionState::EAS_Attacking;
	}
}

void ASlashCharacter::PlayAttackMontage()
{
	UAnimInstance* AnimInstance = GetMesh()->GetAnimInstance();

	if (AnimInstance && AttackMontage)
	{
		AnimInstance->Montage_Play(AttackMontage);
		const int32 Selection = FMath::RandRange(0, 1);
		FName SectionName = FName();

		switch (Selection)
		{
		case 0:
			SectionName = FName("Attack1");
			break;
		case 1:
			SectionName = FName("Attack2");
			break;
		default:
			break;
		}

		AnimInstance->Montage_JumpToSection(SectionName, AttackMontage);
	}
}

void ASlashCharacter::AttackEnd()
{
	ActionState = EActionState::EAS_Unoccupied;
}

bool ASlashCharacter::CanAttack()
{
	return ActionState == EActionState::EAS_Unoccupied &&
		CharacterState != ECharacterState::ECS_Unequipped;
}
```
### Blueprint Validation

- **Getter Validation**: Right-click `Getter SlashCharacter` and convert to a validated get.

### Sounds

- **MetaSounds**: Use MetaSounds (UE5) instead of Sound Cues.
- **Prefix**: Use the `sfx` prefix for sound effects.
- **Notify in AttackMontage**: Add a `Notify` for `sfx_Whoosh`.

### Collision

- **Disable Collision**: Disable item collision when the weapon is equipped to avoid collision with the character. Set it to `nullptr`.

### Slow Motion

- **Key**: Use the `§` key to toggle slow motion.
- **Command**: `Slomo (value)`.

### Colliding Animation

1. **Animation Asset**: Select the animation asset.
2. **Bone Selection**: Choose the bone that is colliding.
3. **Non-Colliding Keyframes**: Add keys for a non-colliding state before and after the collision.
4. **Bone Rotation**: Rotate the bone to the desired location and add keys.

## Section 13 - Weapon Mechanics

### FHitResult
- `HitLocation`: Center of the hit sphere.
- `ImpactPoint`: Actual point of impact on the surface.

### Custom Overlap Function
In `BeginPlay`:
```cpp
WeaponBox->OnComponentBeginOverlap.AddDynamic(this, &AWeapon::OnBoxOverlap);
```

### Set Collision on Weapon Attack
- Use **Anim Notify**:
  - Create a **Notify Track** in the animation.
  - Add notify events to **Enable** and **Disable Collision**.
- **BlueprintCallable Function** in `SlashCharacter`:
  - Call it from `ABP_Echo` via **AnimNotify Event**.

### Unreal Interfaces
- Use **pure virtual function** in interface:
```cpp
virtual void GetHit() = 0;
```

### Add Root Bone to Mixamo Animation in Blender
- Only one animation needs the skin.
- Use **Blender Mixamo Addon**:
  1. Blender → Edit → Preferences → Install Add-on from file → select Mixamo converter (GitHub ZIP).
  2. In Scene Collection → Mixamo Rootbaker:
     - Input: "No Root Bone" folder.
     - Output: "Root Bone" folder.
  3. Advanced Options:
     - Uncheck `Apply Rotation`.
     - Uncheck `Transfer Rotation`.
  4. Use **Batch Convert**.

### Moving Animations
- Enable **Root Motion**:
  - Select the animation asset.
  - Go to Asset Details → Enable "Root Motion".
  - This moves the actor's collision with the animation.

### Actors to Ignore (Trace)
```cpp
for (AActor* Actor : IgnoreActors)
{
    ActorsToIgnore.AddUnique(Actor);
}
```

### Hit Sound
- Use **Sound Attenuation** for spatial/distance effect.
  - Example: In `sfx_HitFlesh` → Source → Sound Attenuation: `SA_HitFlesh`.

### Crash Fix
- **Disable Hot Reloading** in Unreal Editor to avoid instability during code changes.

## Section 14: Breakable Actors

### Fracture Mode
- **Selection mode** → **Fracture mode**
- **Fracture** → **Uniform**
  - **Min/Max Voroni Sites**: How many pieces the mesh should be destroyed into
  - Press **Fracture** twice
  - Check **Use Size Specific Damage Threshold** on the actor

### Breakable Actor
- Add `GeometryCollectionEngine` in build file

#### BreakableActor.h
```cpp
UPROPERTY(VisibleAnywhere)
UGeometryCollectionComponent* GeometryCollection;
```

#### BreakableActor.cpp
```cpp
GeometryCollection = CreateDefaultSubobject<UGeometryCollectionComponent>(TEXT("GeometryCollection"));
SetRootComponent(GeometryCollection);
GeometryCollection->SetGenerateOverlapEvents(true);
```

- Collision → Camera → **Ignore** (to avoid camera colliding with breakable pieces)

### Blueprint Native Event Hit Interface

#### Hitinterface.h
```cpp
UFUNCTION(BlueprintNativeEvent)
void GetHit(const FVector& ImpactPoint);
```

#### Enemy.h
```cpp
virtual void GetHit_Implementation(const FVector& ImpactPoint) override;
```

#### Weapon.cpp
```cpp
if (HitInterface)
{
    // From this
    HitInterface->GetHit(BoxHit.ImpactPoint);
    // To this
    HitInterface->Execute_GetHit(BoxHit.GetActor(), BoxHit.ImpactPoint);
}
```

#### Blueprint Native Event
- Right-click `Event GetHit` → Add call to parent function (C++ implementation)

![Blueprint Native Event](Assets/Documentation/blueprint_native_event.png)

### On Chaos Break Event (Blueprint)
- **On Chaos Break Event** → Set lifespan
- Geometry Collection → Chaos Physics → **Notify Breaks** (checked)

## Section 15 - Treasure

### Sound Effects

- Source: [https://soundimage.org](https://soundimage.org)  
- Credit: Eric Matyas

---

### Fracture

1. **Fracture Workflow**
   - Select mesh → `Fracture Mode` → `Generate New` (save in `Destructables` folder)
   - Adjust **Explode Amount** to preview
   - Click `Fracture` to generate pieces

---

### Breakable Actor

**BreakableActor.h**
```cpp
private:
	UPROPERTY(EditAnywhere, Category = "Breakable Properties")
	TArray<TSubclassOf<ATreasure>> TreasureClasses;

	bool bBroken = false;
```

**BreakableActor.cpp**
```cpp
void ABreakableActor::GetHit_Implementation(const FVector& ImpactPoint)
{
	// prevent infinite loop
	if (bBroken) return; 

	UWorld* World = GetWorld();
	bBroken = true;

	if (World && TreasureClasses.Num() > 0)
	{
		FVector Location = GetActorLocation();
		Location.Z += 75.f;

		const int32 Selection = FMath::RandRange(0, TreasureClasses.Num() - 1);

		World->SpawnActor<ATreasure>(TreasureClasses[Selection], Location, GetActorRotation());
	}
}
```

---

### Niagara System

- **Sprite Renderer** → `Material` (what’s being rendered)
- **Emitter Update** → `Spawn Rate` (emitters spawned per second)
- **Particle Spawn**
  - `Lifetime Mode` → Min/Max
  - `Color Mode` → Color
- **Particle Update**
  - `Particle State` → Uncheck `Kill Particles when Lifetime has elapsed`
  - `Drag` → Wind resistance
  - `Add (+)`:
    - `Vortex Force` → Enable Gravity Force `(0, 0, 50)`
    - Set `Vortex Force Amount` to `50`

---

### Deactivate Niagara Effect

**Item.h**
```cpp
protected:
	UPROPERTY(EditAnywhere, Category = "Visual Effects")
	UNiagaraComponent* EmbersEffect;
```

**Item.cpp**
```cpp
EmbersEffect = CreateDefaultSubobject<UNiagaraComponent>(TEXT("EmbersEffect"));
EmbersEffect->SetupAttachment(GetRootComponent());
```

**Weapon.cpp (on Equip)**
```cpp
if (EmbersEffect)
{
	EmbersEffect->Deactivate();
}
```
## 🛡️ Section 16: Combat

### 📦 Actor Component (Attach to Enemy)
- Attach `HealthBarComponent` to enemy actor.
- Handles health updates and UI logic.

---

### 🧩 Widget Blueprint Setup
- Create a new Widget Blueprint (e.g. `WBP_HealthBar`)
- Add a `Canvas Panel` and a `Progress Bar`:
  - **Anchor**: Center
  - **Alignment**: (0.5, 0.5)
  - **Progress → Percent**: Used to show health visually

---

### 🧱 Widget C++ Setup

#### 🔧 Module Setup
Add `UMG` to your `.Build.cs` file:

```cpp
PublicDependencyModuleNames.AddRange(new string[] { "UMG" });
```

#### 🧩 HealthBar.h
```cpp
public:
	UPROPERTY(meta = (BindWidget))
	UProgressBar* HealthBar;
```

#### 🧩 HealthBarComponent.h
```cpp
private:
	UPROPERTY()
	UHealthBar* HealthBarWidget;
```

#### 🧩 HealthBarComponent.cpp
```cpp
void UHealthBarComponent::SetHealthPercent(float Percent)
{
	if (HealthBarWidget == nullptr)
	{
		HealthBarWidget = Cast<UHealthBar>(GetUserWidgetObject());
	}

	if (HealthBarWidget && HealthBarWidget->HealthBar)
	{
		HealthBarWidget->HealthBar->SetPercent(Percent);
	}
}
```

---

### 🧷 Attach Widget to Enemy
In `BP_Enemy`, add a **Widget Component**:
- `Widget Class`: Your health bar
- `Space`: **Screen**
- `Draw at Desired Size`: ✅ (optional)

This ensures the health bar displays at the enemy's location.

---

### ⚔️ Weapon Equip (Apply Damage)

#### `Weapon.cpp`
```cpp
void AWeapon::Equip(USceneComponent* InParent, FName InSocketName, AActor* NewOwner, APawn* NewInstigator)
{
	SetOwner(NewOwner);
	SetInstigator(NewInstigator);
}
```

---

### 💀 Death Animations

#### Static Death Pose:
- Choose death animation → Right-click → **Create Asset from Current Pose**

#### Blueprint Setup:
- Use `Blueprint Thread Safe Update Animation`
- Freezes animation at death frame (faster than playing a full anim)

#### Animation Montage:
- Set **Blend Time = 0** for instant transitions (no fade/blend)

---

### 🔄 Mixamo → Unreal Workflow (Optional Retargeting)

#### In Blender (Mixamo Add-on):
- Download 1 animation **with skin**, others **without**
- In Add-on:
  - Move from "No Root Bone" to "Root Bone" folder
  - Uncheck:
    - `Apply Rotation`
    - `Transfer Rotation`
  - Use **Batch Convert**

#### In Unreal Engine:
- Import animations as **"Animations Only"**
- Use `SKM_X_Bot` as skeleton
- Open `IK_Retargeter` → Select animations → Export Selected Animations

---

## Section 17: Enemy Behavior

### Nav mesh bounds volume
- **Show in editor**: `P`
- **Show in game**: `§` → `show navigation`
- `BP_Enemy` → Details → Auto Possess AI → `Placed in World or Spawned`
- Update navigation mesh in real time: `Edit → Project Settings → Navigation Mesh → Runtime Generation: Dynamic`
- Navigation cell size/height, area around objects: `Edit → Project Settings → Navigation Mesh → Cell Size / Height`

### Walking enemy animation
- Mixamo → Walking animation → In place (AI nav moving character)
- `BP_Enemy` → Character Movement → `Orient Rotation to Movement` → ✅
- `BP_Enemy` → Self → `Use Controller Rotation Yaw` → ❌

#### C++
```cpp
GetCharacterMovement()->bOrientRotationToMovement = true;
bUseControllerRotationPitch = false;
bUseControllerRotationYaw = false;
bUseControllerRotationRoll = false;
```

### Blend Space 1D
- **Horizontal Axis → Name**: `GroundSpeed`
- **Max Axis Value**: `300`
- **Idle at**: `0`
- **Walk at**: `75`
- **Run at**: `300`

- Use `PaladinIdleWalkRun` in ABP instead of Idle.
- `ABP → EventGraph → Set GroundSpeed`
- `ABP → Add new variable → GroundSpeed`
- `ABP → Blueprint Thread Safe → Property access → Character Movement → Velocity → Get Vector Length XY → Set GroundSpeed`

### Patrol target
```cpp
EnemyController = Cast<AAIController>(GetController());

if (EnemyController && PatrolTarget)
{
    FAIMoveRequest MoveRequest;
    MoveRequest.SetGoalActor(PatrolTarget);
    MoveRequest.SetAcceptanceRadius(15.f);
    FNavPathSharedPtr NavPath;

    EnemyController->MoveTo(MoveRequest, &NavPath);

    TArray<FNavPathPoint>& PathPoints = NavPath->GetPathPoints();

    for (auto Point : PathPoints)
    {
        const FVector& Location = Point.Location;
    }
}
```

- Make sure to include header files and `AIModule` in build.

### Patrol wait time

**Enemy.cpp**
```cpp
GetWorldTimerManager().SetTimer(PatrolTimer, this, &AEnemy::PatrolTimerFinished, WaitTime);
```

**Enemy.h**
```cpp
FTimerHandle PatrolTimer;

void PatrolTimerFinished();
```

### Pawn sensing

**Enemy.cpp**
```cpp
PawnSensingComponent = CreateDefaultSubobject<UPawnSensingComponent>(TEXT("PawnSensingComponent"));
PawnSensingComponent->SightRadius = 4000.f;
PawnSensingComponent->SetPeripheralVisionAngle(45.f);

// In BeginPlay
if (PawnSensingComponent)
{
    PawnSensingComponent->OnSeePawn.AddDynamic(this, &AEnemy::PawnSeen);
}
```

- **Tags**
```cpp
Tags.Add(FName("SlashCharacter"));
```

- **Lighter than casting**
```cpp
void AEnemy::PawnSeen(APawn* SeenPawn)
{
    if (SeenPawn->ActorHasTag(FName("SlashCharacter")))
    {
        UE_LOG(LogTemp, Warning, TEXT("AEnemy::PawnSeen: %s"), *SeenPawn->GetName());
    }
}
```

### Origin point for meshes
Some weapons can have different origin points and will look off when attaching to socket. We can fix this in Blender.

- In UE5: Go to mesh → Asset Action → Export → `Assets/Meshes/NeedAlterations` → Uncheck Level of Detail and Collision
- In Blender: Import asset → Select all with `A` → Object Mode → Edit Mode
    - Rotate 180° on Y-axis: `R → Y → 180`
    - Move origin: `G → Z`
    - Scale: `S`

Export as `.fbx`, then re-import in UE5 and **disable material import**.

## Section 18: Enemy Attacks

### Inheritance
- Replace `#include "GameFramework/Character.h"` with `#include "BaseCharacter.h"` in characters.
- Inherited functions/variables cannot have `UPROPERTY` in child class.

### Cannot Save Bug
- Delete cache and restart project.

### Fixing Clipping in an Animation
1. Open the animation asset.
2. Identify the bone that needs adjustment.
3. Add a keyframe at the beginning where the pose looks correct.
4. Add another keyframe at the end where the pose is still correct.
5. Navigate to the section where the clipping occurs.
6. Adjust the bone to fix the issue.
7. Add a keyframe at that point.
8. Play the animation to review the changes.

### Max Enum Value (C++ Example)
```cpp
TEnumAsByte<EDeathPose> Pose(Selection);

if (Pose < EDeathPose::EDP_MAX)
{
    DeathPose = Pose;
}
```

### Avoid Character Moving After Death (C++ Example)
```cpp
GetCharacterMovement()->bOrientRotationToMovement = false;
```

### Death Sound Implementation
1. Download sound → `Assets/Sound`
2. Import sound effect → Create MetaSound → Add random pitch

## Section 19: Smarter Enemies
### IK Rig
- Skeletal mesh must have same pose (T)
- Download mesh with T pose -> Root bone conversion in Blender -> Rename new fbx to the same as the current -> Paste in
- Create IK Rig -> Set Hips (same as X_BOT) as root bone
- Add chains according to X_BOT

### IK Retargeter
- Add Paladin as source and Echo as target
- Select edit mode -> Select target -> Align all bones
- Select Root (chain) -> Translation Mode -> Globally Scaled

### Weapon Collision
```cpp
SetWeaponCollisionEnabled(ECollisionEnabled::NoCollision);
```
- If get hit interrupts attack montage and still have collision enabled

### Motion Warping
- Edit -> Plugins -> Motion Warping
- BP_BaseEnemy -> Add Motion Warping component
- BP_BaseEnemy -> SetWarpTarget custom event -> Validated Get -> Motion Warping component -> Add or Update Warp Target from Transform -> CombatTarget -> Get Actor Transform
- Attack animation montage -> New notify track -> Add notify state -> Motion Warping
- Anim Notify -> Config -> Root Motion Modifier -> Skew Warp -> Warp Target Name -> CombatTarget
- Attack animation montage -> New notify track -> UpdateTranslationTarget -> Add notify state

- ABP -> UpdateTranslationTarget Event -> Set Warp Target

### Echo Motion Warp
- TargetDetectionSphere->OnComponentBeginOverlap
- Cast to enemy and set to combat target

## Section 20: Echo's Attributes

### Textures
- Set all textures to 2D → Highlight → Right-click → Asset Actions → Bulk Edit via Property Matrix → Compression → Compression Settings → UserInterface2D

### Widget Blueprint
- Set image to texture size → Size To Content
- See parent class in blueprint → top right of details
- Bind C++ with WBP. Make sure they have the same name

Example:
```cpp
UPROPERTY(meta = (BindWidget))
UProgressBar* StaminaProgressBar;
```

## Section 21: Souls and Stamina

### Spawn Soul

**Enemy.cpp**
```cpp
void AEnemy::SpawnSoul()
{
	UWorld* World = GetWorld();

	if (World && SoulClass && AttributeComponent)
	{
		ASoul* SpawnedSoul = World->SpawnActor<ASoul>(SoulClass, GetActorLocation(), GetActorRotation());

		if (SpawnedSoul)
		{
			SpawnedSoul->SetSouls(AttributeComponent->GetSouls());
		}
	}
}
```

**Enemy.h**
```cpp
UPROPERTY(EditAnywhere, Category = "Combat")
TSubclassOf<ASoul> SoulClass;
```

**Soul.h**
```cpp
FORCEINLINE void SetSouls(int32 Amount) { Souls = Amount; }
```

**AttributeComponent.h**
```cpp
private:
	UPROPERTY(EditAnywhere, Category = "Actor Attributes")
	int32 Souls;

public:
	FORCEINLINE int32 GetSouls() const { return Souls; }
```

---

### Regen Stamina

**AttributeComponent.cpp**
```cpp
UAttributeComponent::UAttributeComponent()
{
	PrimaryComponentTick.bCanEverTick = false;
}

void UAttributeComponent::RegenStamina(float DeltaTime)
{
	CurrentStamina = FMath::Clamp(CurrentStamina + StaminaRegenRate * DeltaTime, 0.f, MaxStamina);
}
```

**SlashCharacter.cpp**
```cpp
ASlashCharacter::ASlashCharacter()
{
	PrimaryActorTick.bCanEverTick = true;
}

void ASlashCharacter::Tick(float DeltaTime)
{
	if (AttributeComponent && SlashOverlay)
	{
		AttributeComponent->RegenStamina(DeltaTime);
		SlashOverlay->SetStaminaBarPercent(AttributeComponent->GetStaminaPercent());
	}
}
```

## Section 22: Multiple Types of Enemies

### Animation Blueprint → Template
- Blend Poses
  - Sequence Player
  - Dead 1, Dead 2, Dead 3…
  - Promote to variables
  - Generic death animations for enemies
- Make child class of template
  - Set Dead 1–5 in blueprint

### Raptor Enemy
- Blend Space 1D
  - Horizontal Axis → Speed
  - Shift‑drag idle animation to the start
  - Shift‑drag run animation to the end

### Not Able to Hit Enemy
- Double-check physics asset
  - Mesh → Physics → Physics Asset
  - Right‑click mesh → Create and Assign Physics Asset

### Ragdoll Death
- Set Die as BlueprintNativeEvent
- In Blueprint: Event Die → Set Simulate Physics = true → Set Collision Enabled = Physics Only

### Multiple Weapon Sockets
- In attack montage → Add notify to animation
- Set event in ABP
- Use Equipped Weapon → Attach Actor To Component with WeaponSocket name
