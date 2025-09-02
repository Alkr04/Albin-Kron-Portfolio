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

The game was originally pitched as a grid based 2 player turn based Party/Strategy game with plenty of weapons. Originally I was drawn in by the turn and grid based idea of the game but those Ideas were discarded shortly before proper development began due to being too complex and time consuming. But during the design meetings we also expanded a bit on the idea of multiple weapons which ended up capturing my imagination. So while we were still concepting the game I was also starting to imagine how the game could have multiple weapons without spending too much time on each individual weapon.

Half a year before that, one friend recommended that I should learn scriptable objects due to how useful they are. And I tested them out on a project and found them to be really good for creating different settings for the same script. Thus when I was planning how the weapon system was going to work I quickly concluded that scriptable objects were going to be a good fit. so I created one gun script that holds all the functions for the gun. and a scriptable object for the gun's settings.

When a player picks up a weapon it assigns all the properties of the weapon scriptable object to the player's weapon and adds the proper functions to unity events.


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

The Shoot function starts when a player fires a weapon. It determines if the player can fire, what fire mode the weapon uses, how many shots per key press, if it fires like a shot gun based on the scriptable object stored as "gun". then it calls the bullet pool to send a bullet.

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

Bullets work in a similar manner to how picking up new guns work. Except that it has 3 separate events that the different functions use. one when they have just been fired, when while flying and one when it hits something. but nothing currently uses the one for when it has just been fired. There is also a 4th event if the weapon is an area of effect.


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

We were supposed to have just 1 bullet prefab and change everything about it including its looks and VFX when it was fired but due to time constraints and lack of knowledge we did not manage to implement that. Instead each bullet has its own prefab making this system redundant in the current version other than the ease of editing the bullet's attributes. The ground work is there if we want to add it in the future.

According to the other developers the system is quite easy to use and perfect for making plenty of guns.


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
