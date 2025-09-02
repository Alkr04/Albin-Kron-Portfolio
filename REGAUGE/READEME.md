# REGAUGE

System Programer

Duration: 04/2025 - 06/2025

Engine: Unity

Genre: Couch Co-op, Shooter

Team: 3 Programers, 4 Artists

<table>
  <tr>
    <td><img src="Gifs/4PlayerGameplay.gif" /></td>
    <td><img src="Gifs/SpinnyPlate.gif" /></td>
  </tr>
</table>

## My contribution

### Weapon and bullet system


I was the primary developer behind the weapon system. 

The game was originaly pitched as a grid based 2 player turn based Party/Strategy game whit plenty of weapons. originaly I was drawn in by the turn and grid based idea of the game but those Ideas were discarded shortly before proper development began do to being to complex and time consuming. But during the designe meatings we also expanded a bit on the idea of multeple weapons whitch ended up capturing my imagination. So while we were still consepting the game I was also starting to imagine how the game culd have multeple weapons whitout spending to much time on each indevidual weapon.

half a year before that one frend recomended that I shuld lern scripteble objects do to how usefull they are. And I tested them out on a project and foud them to be realy good for creating diferent setings for the same script. Thus when I was planing how the wapon system was going to work I quicly concluded that scripteble objects were going to be a good fit. so I created one gun script that holds all the functions for the gun. and a scripteble object for the gun's setings.

When a player picks up a weapon it assignes all the properties of the weapon skripteble object to the players weapon and adds the proper funktions to unity events.

<details>
  
  <summary> picking up gun </summary>

```csharp
public void NewGun(gunBase newgun)
    {
        if (!playerCharacter) { return; }
        fiering = false;
        if (gun.loopingAnimation) { animator.SetBool("IsFiring", fiering); }
        gun = newgun;
        source.loop = false;
        AudioClip clip = GameManager.Instance.AudioModulator(gun.equipSound, source, gun.equipMinPitch, gun.equipMaxPitch, gun.equipVolume);
        if (clip) { source.PlayOneShot(clip); }
        currentAmmo = gun.bulletCapasity;
        CheckForGun();

        if (laserSight)
        {
            laserSight.laserIsOn = gun.laserSight;
            laserSight.laserRange = gun.laserSightRange;
        }
        if (gun.shotGun)
        {
            fire -= Scatter;
            fire -= Fire;
            fire += Scatter;
        }
        else
        {
            fire -= Fire;
            fire -= Scatter;
            fire += Fire;
        }
        if (gun.doesItHaveRecoil)
        {
            fire -= Recoil;
            fire += Recoil;
        }
        else
        {
            fire -= Recoil;
        }
    }
```
  
</details>

the shoot funktion starts when a player fiers a weapon. It determins if the player can fire, what fire mode the wapon uses, how meny shots per key press, if it fires like a shot gun based on the scripteble object stored as "gun". then it calls the bullet pool to send a bullet.

<details>
  <summary>shoting</summary>

  ```csharp
public void Shoot()
    {
        if (disableGun) { return; }

        if (currentAmmo > 0 && canFire && !keyDown && playerCharacter.playerMovement.CheckDashIntractability())
        {
            currentAmmo -= gun.consumed;
            SnapRotation();
            CheckForGun();
            canFire = false;
            fireMode[gun.fireMode.ToString()].Invoke();
        }
        else if (!keyDown && currentAmmo <= 0)
        {
            keyDown = true;
            DepressKey();
        }
    }

void SingleShot()
    {
        keyDown = true;
        StartCoroutine(Burst());
        StartCoroutine(Cooldown(gun.fireRate / firerateModifier));
    }
    void AutoShot()
    {
        StartCoroutine(Burst());
        StartCoroutine(Cooldown(gun.fireRate / firerateModifier));
    }

IEnumerator Burst()
    {
        for (int i = 0; i < gun.burstAmount; i++)
        {
            fire.Invoke();
            if (!fiering)
            {
                StartCoroutine(FireAudio());
            }
            yield return new WaitForSeconds(gun.timeBetweneBurstShots);

        }
        if (currentAmmo <= 0)
        {
            NewGun(baseGun);
        }
    }

public void Fire()
    {
        Vector3 firePos = HandleFirePos();
        Quaternion rotation = Quaternion.LookRotation(Vector3.up, GetAimDirection());
        Vector3 euler = rotation.eulerAngles;

        GameObject bullet = PoolManager.poolInstance.GiveBullet(gun.bullet, firePos, Quaternion.Euler(new Vector3(euler.x, euler.y, euler.z + UnityEngine.Random.Range(-gun.spread, gun.spread))), gun.bulletBase,
            transform.root.GetComponent<Collider>(), this);

        StartMuzzleFlash(firePos, rotation);
        StartShootAnim();
    }

void Scatter()
    {
        float temp = gun.spread;

        Quaternion rotation = Quaternion.LookRotation(Vector3.up, GetAimDirection());
        Vector3 euler = rotation.eulerAngles;
        Vector3 firePos = HandleFirePos();

        for (int i = 0; i < gun.pellets; i++)
        {
            if (temp >= 0)
            {
                temp -= (gun.spread / gun.pellets) * 2 * UnityEngine.Random.Range(0.5f, 1.5f);
            }
            temp *= -1;

            //GameObject bullet = Instantiate(gun.bullet, firePos, Quaternion.Euler(new Vector3(euler.x, euler.y, euler.z + temp)));
            GameObject bullet = PoolManager.poolInstance.GiveBullet(gun.bullet, firePos, Quaternion.Euler(new Vector3(euler.x, euler.y, euler.z + temp)), gun.bulletBase, transform.root.GetComponent<Collider>(), this);
            //InitializeBullet(bullet, firePos);

        }


        StartMuzzleFlash(firePos, rotation);
        StartShootAnim();
    }
```
</details>

Bullets work in a similar maner to how picking up new guns work. Exept that it has 3 seperet events that the diferent funktions use. one when they have just ben fiered, when while flying and one when it hits something. but nothing curently uses the one for when it has just ben fiered. there is also a 4th event if the weapon is a area of effect.

<details>
<summary> bullets</summary>

```csharp
void SubscribeComponents()
    {
        if (bullet.bounce)
        {
            hit += Bounce;
        }
        if (bullet.aoe)
        {
            hit += AreaOfEffect;
        }
        if (bullet.aoeKnockBack)
        {
            aoeEffect += AOEKnockBack;
        }
        if (!bullet.aoeDamage)
        {
            hit += Damage;
        }
        else
        {
            aoeEffect += CallDamagePlayer;
        }
        if (bullet.destroyOnCollition)
        {
            hit += DestroyBullet;
        }
        if (bullet.magnetism)
        {
            flying += Magnetize;
        }
    }

```
  
</details>

We were supose to have just 1 bullet prefab and change every thing about it including it's looks and VFX when it was fired but do to time constraints and lack of knolage we did not manage to implement that. Instead each bullet has it's own prefab making this system redundent in the curent version other then the ease of edeting the bullet's atributes. The ground work is there if we want to add it in the future.

Acording to the other developers the system is quite easy to use and perfect for making plenty of guns.

<table>
  <trt>
    <td><img src="Pictures/ExampleGunSettings.png"/></td>
    <td><img src="Pictures/ExampleBulletSettings.png"/><td>
  </trt>
</table>

### My other contributions

* the pilote doors

* the bullet pools

* the audio system 
