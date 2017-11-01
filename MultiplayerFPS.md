# Scripts Files

## GameManager.cs

### Location: GameManager(Empty GameObject)

### Content:
* Attributes
  * public static GameManager instance
  * [SerializeField] sceneCamera
  * OnPlayerKilledCallback
  * private const string PLAYER_ID_PREFIX
  * private static Dictionary<string, Player> players
* Function
  * void Awake(): generate instance
  * void SetSceneCameraActive(bool isActive): set the sceneCamera active or not
  * void RegisterPlayer(string netID, Player player): set playerID, and it to the dictionary
  * void UnRegisterPlayer(string playerID): remove the player from the dictionary
  * Player GetPlayer(string playerID): return a certain player
  * Player[] GetAllPlayers(): return all players

## PlayerMotor.cs

### Location: Player prefabs

### Content:
* Attributes
  * [SerializeField] private Camera cam
  * private Vector3 velocity
  * private Vector3 rotation
  * private float cameraRotationX
  * private float currentCameraRotationX
  * private Vector3 thrusterForce
  * [SerializeField] private float cameraRotationLimitation
  * private Rigidbody rb
* Functions
  * void Start(): set rigidbody
  * void Move(Vector3 velocity): set velocity
  * void Rotate(Vector3 rotation): set rotation
  * void RotateCamera(float cameraRotationX): set cameraRotationX
  * void ApplyThruster(Vector3 thrusterForce): set thrusterForce
  * void FixedUpdate(): call PerformMovement() and PerformRotation()
  * void PerformMovement(): if velocity has value, move rb position. if thrusterForce has value, rb add force
  * void PerfromRotation(): rb rotation, cam rotation
    (The camera in player will follow rigid body to rotate horizontally but still need to rotate vertically)

## PlayerController.cs

### Location: Player prefabs

### Content:
* Attributes:
  * [SerializeField] float speed
  * [SerializeField] float lookSensitivity
  * [SerializeField] float thrusterForce
  * [SerializeField] float thrusterFuelBurnSpeed
  * [SerializeField] float thrusterFuelRegenSpeed
  * private thrusterFuelAmount
  * [SerializeField] LayerMask environmentMask
  * [SerializeField] float jointSpring
  * [SerializeField] float jointMaxForce
  * PlayerMotor motor
  * ConfigurableJoint joint
  * Animator animator
* Functions:
  * float GetThrusterFuelAmount(): return thrusterFuelAmount
  * void Start(): get motor, joint, animator. Call SetJointSettings()
  * void Update(): 
  	if user press pause, lock cursor, set motor to zero
  	Set joint. Calculate velocity according to Input and call motor.Move. Calculate rotation and call motor.Rotate. Calculate camera rotation and call motor.RotateCamera
  	Calculate thrusterForce and thrusterFuel according to users press "Jump" or not
  * void SetJointSettings(float jointSpring): set jointDrive

## PlayerSetup.cs

### Location: Player prefabs

### Content:
* Attributes:
  * [SerializeField] Behaviour[] componentsToDisable
  * [SerializeField] string romoteLayerName
  * [SerializeField] string dontDrawLayerName
  * [SerializeField] GameObject playerGraphics
  * [SerializeField] GameObject playerUIPrefab
  * [HideInInspector] GameObject playerUIInstance
* Functions:
  * void Start(): if !isocalPlayer, call DisableComponents(), AssignRemoteLayer()
  else, call SetLayerRecursively(), Instantiate playerUIInstance,
  ui.SetPlayer, Player.PlayerSetup()
  Call CmdSetUsername()
  * [Command] void CmdSetUsername(string playerID, string username): get certain player and set player's username
  * void SetLayerRecursively(GameObject obj, int newLayer): recursively set layer name
  * override void OnStartClient(): Call GameManager.RegisterPlayer()
  * void RegisterPlayer(): set tranform.name
  * void DisableComponents(): let all elements in componentsToDisable enabled = false
  * OnDisable(): Destroy(playerUIInstance), for localPlayer set scene camera active, unregisterPlayer from GameManager

## PlayerShoot.cs

### Location: Player prefabs

### Content:
* Attributes:
  * const sring PLAYER_TAG
  * PlayerWeapon currentWeapon
  * WeaponManager weaponManager
  * [SerializeField] Camera cam
  * [SerializeField] LayerMask mask
* Functions:
  * void Start(): set weaponManager
  * void Update(): set currentWeapon, according to the fireRate, call Shoot()
  * [Command] void CmdOnShoot(): call RpcDoShootEffect()
  * [Command] void CmdOnHit(Vector3 pos, Vector3 normal): call RpcDoHitEeffect()
  * [ClientRpc] void RpcDoShootEffect(): play muzzleFlash
  * [ClientRpc] void RpcDoHitEffect(Vector3 pos, Vector3 normal): add hitEffect and destroy it after 2 seconds
  * [Client] void Shoot(): calculate bullets and send raycastHit
  * [Command] void CmdPlayerShot(string playerID, int damage, string sourceID): get player who was hitten and call player.RpcTakeDamage()

## Player.cs

### Location: Player prefabs

### Content:
* Attributes:
  * [SyncVar] bool _isDead
  * [SerializeField] int maxHealth
  * [SyncVar] int currentHealth
  * [SyncVar] string username
  * int kills
  * int deaths
  * [SerializeField] Behaviour[] disableOnDeath
  * bool[] wasEnabled
  * [SerializeField] GameObject deathEffect
  * [SerializeField] GameObject spawnEffect
  * bool firstSetup
* Functions
  * float GetHealthPercentage(): return currentHealth / maxHealth
  * void PlayerSetup(): if it's localPlayer, GameManger's scene camera set active, PlayerSetup's UI instance set active. Call CmdBroadCastNewPlayerSetup()
  * [Command] void CmdBroadCastNewPlayerSetup(): Call RpcSetupPlayerOnAllClients()
  * [ClientRpc] void RpcSetupPlayerOnAllClients(): if it's firstSetup, set wasEnabled list. Call SetDefaults()
  * void SetDefaults(): set disableOnDeath(Scripts) as wasEnabled, set disableGameObjectsOnDeath as active, set collider enable, Instantiate spawnEffect and destroy it after 3 seconds
  * IEnumerator Respawn(): set player to a spawnPoint and call PlayerSetup()
  * [ClientRpc] void RpcTakeDamag(int damage, string sourceID): set currentHealth, if it's less than or equal to zero, call Die()
  * void Die(string sourceID): set GameManager's onPlayerKilledCallback, disable disableOnDeath and disableGameObjectsOnDeath and collider. Show deathEffect and destroy it after 3 seconds. If it's local player, set scene camera active and playerUIInstance in PlayerSetup false. Call Respawn()

## WeaponManager.cs

### Location: Player prefabs

### Content:
* Attributes:
  * [SerializeField] string weaponLayerName
  * [SerializeField] Transform weaponHolder
  * [SerializeField] PlayerWeapon primaryWeapon
  * PlayerWeapon currentWeapon
  * WeaponGraphics currentGraphics
  * bool isReloading
* Functions:
  * void Start(): Call EquipWeapon()
  * PlayerWeapon GetCurrentWeapon(): return current weapon
  * WeaponGraphics GetCurrentGraphics(): return current Graphics
  * void EquipWeapon(PlayerWeapon weapon): set weapon gameObject instance's position, graphics
  * void Reload(): call Reload-Coroutine()
  * IEnumerator Reload_Coroutine(): wait a reloadTime and set bullets to the maxBullets
  * [Command] void CmdOnReload(): Call RpcOnReload()
  * [ClientRpc] void RpcOnReload(): Awake Reloading animation

## PlayerScore.cs

### Location: Player prefabs

### Content:
* Attributes:
  * int lastKills
  * int lastDeath
  * Player player
  * void Start(): set player and call SyncScoreLoop()
  * void OnDestroy(): Call SyncNow()
  * IEnumerator SyncScoreLoop(): while (true), call SyncNow() every 5 seconds
  * void SyncNow(): Use UserAccountManager.instance.GetData() to call OnDataReceived
  * void OnDataReceived(string data): Update player's scores to the database. Player.kills / Player.deaths record the kills and deaths in one game. Update scores in database every 5 seconds by calculating player.kills - last 5 seconds kills

## WeaponGraphics.cs

### Location: Sci-Fi Automatic prefabs

### Content:
* Attributes:
  * ParticleSystem muzzleFlash
  * GameObject hitEffect

##  PlayerUI.cs

### Location: PlayerUI prefabs

### Content:
* Attributes:
  * [SerializeField] RectTransform thrusterFuelFill
  * [SerializeField] RectTransform healthBarFill
  * [SerializeField] GameObject pauseMenu
  * [SerializeField] GameObject scoreboard
  * [SerializeField] Text ammoText
  * Player player
  * PlayerController controller
  * WeaponManager weaponmanager
* Functions:
  * void SetPlayer(Player player): set player, controller and weaponmanager
  * void Start(): set pauseMenu.IsOn = false
  * void Update(): Call SetFuelAmount(), SetHealthAmount() and SetAmmoAmount(). If pressing KeyCode.Escape call TogglePauseMenu(), if pressing KeyCode.Tab, call scoreboard.SetActive(), if pressing KeyCode.Tab, call scoreboard.SetActive().
  * void TogglePauseMenu(): puaseMenu.setActive()
  * void SetFuelAmount(float amount): set thrusterFuelFilld.localScale
  * void SetHealthAmount(float amount): set healthBarFill.localScale
  * void SetAmmoAmount(int amount): set ammoText.text

##  PauseMenu.cs

### Location: PauseMenu prefabs

### Content:
* Attributes:
  * static bool IsOn = false
  * NetworkManager networkmanager
* Functions:
  * void Start(): set networkManager
  * void LeaveRoom(): Drop connection and stop host

##  Scoreboard.cs

### Location: Scoreboard prefabs

### Content:
* Attributes:
  * [SerializeField] GameObject playerScoreboardItem
  * [SerializeField] Transform palyerScoreboardList
* Functions():
  * void OnEnable(): set player scores board item
  * void OnDisable(): Loop to destroy all item in the list

##  Killfeed.cs

### Location: Killfeed prefabs

### Content:
* Attributes:
  * [SerializeField] GameObject killfeedItemPrefab
  * void Start():  GameManager.instance.onPlayerKilledCallback += OnKill;
  * void OnKill(string player, string source): Set KillfeedItem
  
##  PlayerScoreboardItem.cs

### Location: PlayerScoreboardItem prefabs

### Content:
* Attributes:
  * [SerializeField] Text usernameText
  * [SerializeField] Text killText
  * [SerializeField] Text deathText
* Functions:
  * void Setup(string username, int kills, int deaths): set usernameText, killText and deathText

## RoomListItem.cs

### Location: RoomListItem prefabs

### Content:
* Attributes:
  * delegate void JoinRoomDelegate(MatchInfoSnapshot match)
  * JoinRoomDelegate joinRoomCallback
  * [SerializeField] Text roomNameText
  * MatchInfoSnapshot match
* Functions:
  * void Setup(MatchInfoSnapshot match, JoinRoomDelegate joinRoomCallback): set match, joinRoomCallback and roomNameText
  * void JoinRoom(): joinRoomCallback.Invoke(match)

## UserAccountManager.cs

### Location: User Account Manager(Empty GameObject)

### Content:
* Attributes:
  * static UserAccountManager instance
  * string loggedInSceneName
  * string loggedOutSceneName
  * static bool isLoggedIn
  * delegate void OnDataReceivedCallback(string data)
  * string playerUsername
  * string playerPassword
* Functions:
  * void Awake(): Generate an instance, and DontDestroyOnLoad()
  * void LogIn(): SceneManager.LoadScene(loggedInSceneName), set isLoggedIn true
  * void LogOut(): SceneManager.LoadScene(loggedOutSceneName)
  * void GetData(OnDataReceivedCallback onDataReceived): Call sendGetDataRequest()
  * IEnumerator sendGetDataRequest(string username string password, OnDataReceivedCallback onDataReceived): Get data by using DCF.GetUserData(username, password), onDataReceived.Invoke(response)
  * void SendData(string data): Call sendSendDataRequest()
  * sendSendDataRequest(string username, string password, string data): send data by using DCF.SetUserData(username, password, data)

## LoginMenu.cs

### Location: LoginMenu(Empty GameObject)

### Content:
* A copy from database control. Modified to correspond with UserAccountManager

## HostGame.cs

### Location: HostGame(Empty GameObject)

### Content:
* Attributes:
  * [SerializeField] uint roomSize
  * string roomName
  * NetworkManager networkManager
* Functions:
  * void Start(): initialize networkManger and call networkManager.StartMarchMaker()
  * void SetRoomName(string name): set roomName
  * void CreateRoom(): Call networkManager.matchMaker.CreateMatch()

## JoinGame.cs

### Location: JoinGame(Empty GameObject)

### Content:
* Attributes:
  * List<GameObjet> roomList
  * [SerializeField] Text status
  * [SerializeField] GameObject roomListItemPrefab
  * [SerializeField] Transfrom roomListParent
  * NetworkManager networkManager
* Functions:
  * void Start(): Initialize networkManager and call networkManager.StartMatchMaker(). Call RefreshRoomList()
  * void RefreshRoomList(): Call ClearRoomList(). Call networkManager.matchMaker.ListMatches(). Set status as "Loading..."
  * void OnMatchList(bool success, string extendedInfo, List<MatchInfoSnapshot> matchList): Initialize roomListItem. Call roomListItem.Setup(). Add it to the roomList
  * void ClearRoomList(): Traverse roomList and destroy every item
  * void JoinRoom(MatchInfoSnapshot match): Call networkManager.matchMaker.JoinMatch(). StartCoroutine(WaitForJoin())
  * IEnumerator WaitForJoin(): Call ClearRoomList(). Initialize countdown to 10. Minus 1 per second.  When countdown equals 0, set status to "Failed to connect". Call networkManager.matchMaker.DropConnection(). Call RefreshRoomList()

