Main.unity
├─ Terrain
├─ SafeZone (center)
├─ GameManager
│  └─ PlayerSpawner
├─ Buildings
│  ├─ House1
│  ├─ House2
│  └─ Barricades
├─ WeaponPickups
│  ├─ Pickup1
│  ├─ Pickup2
│  └─ Pickup3
├─ Player (spawned at runtime)
├─ Canvas
│  ├─ HealthSlider
│  └─ AmmoText
using UnityEngine;
using Photon.Pun;

public class WeaponPickup : MonoBehaviour
{
    public GameObject weaponPrefab;

    void OnTriggerEnter(Collider other)
    {
        if (!other.CompareTag("Player")) return;

        // Give weapon to player (simplified)
        var playerShoot = other.GetComponent<PlayerShoot>();
        if (playerShoot != null)
            playerShoot.bulletPrefab = weaponPrefab;

        // Destroy pickup after collection
        PhotonNetwork.Destroy(gameObject);
    }
}
FightForLifePackage/
├─ Assets/
│  ├─ Scenes/
│  │  └─ Main.unity
│  ├─ Scripts/
│  │  ├─ Player/
│  │  │  ├─ PlayerMovement.cs
│  │  │  ├─ PlayerShoot.cs
│  │  │  ├─ PlayerHealth.cs
│  │  │  └─ LocalCamera.cs
│  │  ├─ GameManager/
│  │  │  ├─ PlayerSpawner.cs
│  │  │  └─ SafeZone.cs
│  │  ├─ Weapons/
│  │  │  ├─ Weapon.cs
│  │  │  └─ WeaponPickup.cs
│  │  └─ UI/
│  │     ├─ HealthUI.cs
│  │     └─ AmmoUI.cs
│  ├─ Prefabs/
│  │  ├─ Player.prefab
│  │  ├─ Bullet.prefab
│  │  ├─ SafeZone.prefab
│  │  ├─ Weapon.prefab
│  │  └─ Buildings/
│  │     ├─ House1.prefab
│  │     ├─ House2.prefab
│  │     └─ Barricade.prefab
│  ├─ Materials/
│  ├─ Textures/
│  └─ Photon/
│     └─ PhotonServerSettings.asset
using UnityEngine;
using Photon.Pun;

public class Bullet : MonoBehaviour
{
    public float damage = 20f;

    void OnCollisionEnter(Collision collision)
    {
        PhotonView pv = collision.gameObject.GetComponent<PhotonView>();
        if (pv != null)
        {
            pv.RPC("TakeDamage", RpcTarget.All, damage);
        }
        Destroy(gameObject);
    }
}
using UnityEngine;

public class PlayerMovement : MonoBehaviour
{
    public float speed = 5f;
    public float jumpHeight = 2f;
    private CharacterController controller;
    private Vector3 velocity;
    private bool isGrounded;

    public float gravity = -9.81f;

    void Start()
    {
        controller = GetComponent<CharacterController>();
    }

    void Update()
    {
        isGrounded = controller.isGrounded;
        if (isGrounded && velocity.y < 0)
            velocity.y = -2f;

        float x = Input.GetAxis("Horizontal");
        float z = Input.GetAxis("Vertical");

        Vector3 move = transform.right * x + transform.forward * z;
        controller.Move(move * speed * Time.deltaTime);

        if (Input.GetButtonDown("Jump") && isGrounded)
            velocity.y = Mathf.Sqrt(jumpHeight * -2f * gravity);

        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);
    }
}
``````````````using UnityEngine;

public class PlayerShoot : MonoBehaviour
{
    public GameObject bulletPrefab;
    public Transform shootPoint;
    public float bulletSpeed = 20f;

    void Update()
    {
        if (Input.GetButtonDown("Fire1"))
        {
            GameObject bullet = Instantiate(bulletPrefab, shootPoint.position, shootPoint.rotation);
            Rigidbody rb = bullet.GetComponent<Rigidbody>();
            rb.velocity = shootPoint.forward * bulletSpeed;
            Destroy(bullet, 3f);
        }
    }
}
using UnityEngine;

public class SafeZone : MonoBehaviour
{
    public Transform zone;
    public float shrinkSpeed = 0.5f;
    public float minSize = 10f;

    void Update()
    {
        if (zone.localScale.x > minSize)
        {
            zone.localScale -= Vector3.one * shrinkSpeed * Time.deltaTime;
        }
    }
}
