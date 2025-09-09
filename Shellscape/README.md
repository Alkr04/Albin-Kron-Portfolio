# Shellscape
Enemy Programer

Duration: 11/2024 - 01/2025

Engine: Unity

Genre: First Person Shoter

Team: 3 Programers, 3 Artists

<table>
  <tr>
    <td><img src="Gifs/Shellscape1.gif" /></td>
  </tr>
</table>

# My contribution
I was primarily in charge of the enemy


The project was plagued by problems. Most stemmed from bad planning and extreme overscoping. But the game came together in the end. I did learn a lot from this project due to all the problems and I think they helped me become a better programmer.


In the original concept for the game there were supposed to be 3 bosses. One for each programmer and since I was supposed to make the first one I tried to create a modular system that the others could use for their bosses. And to do that I figured that utilizing inheritance would be a good idea. And due to the boss having multiple phases I decided to use a state machine to keep track of the phases.


<details>

<summary>State machine</summary>

  ```csharp
private void Awake()
    {
        phase = phases[activePhase];
        health = GetComponent<Enemi_health>();
        enemy = GetComponent<Base_enemy>();
    }

    public void Update()
    {
        if (health.currentHP < health.MaxHP * phase3HPPercent && phase == phases[1] && !PhaseSwitch)
        {
            activePhase = 2;
            StartCoroutine(wait(4, activePhase));
        }
        else if (health.currentHP < health.MaxHP * phase2HPPercent && phase == phases[0] && !PhaseSwitch)
        {
            activePhase = 1;
            StartCoroutine(wait(4, activePhase));
        }
        phase.phase();
    }

    IEnumerator wait(float time, int NewPhase)
    {
        enemy.stop();
        enemy.atta = false;
        PhaseSwitch = true;
        health.source.PlayOneShot(PhaseSwitchSound, 0.5f);
        yield return new WaitForSeconds(time);
        phase = phases[NewPhase];
        PhaseSwitch = false;
        enemy.atta = true;
        enemy.start();
    }
```
</details>

The phases themselves inherited from a script that held all the boss's attacks which inherits from a BasePhase script.

It was made early on when we were still planning on having multiple bosses and for that purpose it would have worked fine since the BasePhase script could hold common functions for all bosses. While the attack ones could hold there more specialised attack funktions. and the actual phase scripts could hold the logic of when things happen.


<details>

<summary>BasePhaseScript</summary>
  
```csharp
 public void resetpositon()
    {
        agent.SetDestination(transform.position);
    }
    public void Move()
    {
        agent.SetDestination(player.transform.position);
    }

```
  
</details>

<details>
<summary>EnemyAttacks</summary>
  
```csharp
  
private void Awake()
    {
        UrchinSpawnerScript = FindObjectOfType<UrchinSpawner>();
        enemy = GetComponentInParent<Base_enemy>();
        agent = GetComponentInParent<NavMeshAgent>();
        weekpoint = transform.parent.GetComponentInChildren<weekpoint>();
    }
    
 
    public IEnumerator cooldown(float t)
    {
        
        enemy.atta = true;
        attack.parent.start();
        attack.still = false;
        yield return null;
    }

    public IEnumerator dublewave(int amount)
    {
        float firstWaveDelay = 2.9f;
        float secondWaveDelay = 1.0f;
        stunable = true;

        if (amount == 1)
        {
            SoundcueHandler.PlayWaveCue();
            StartCoroutine(enemy.weakPoint.SingleShockwave());
            enemy.GetComponentInChildren<MantisAnimator>().anim.SetTrigger("Shockwave");
        }
        else if(amount > 1)
        {
            SoundcueHandler.PlayDoubleWaveCue();
            StartCoroutine(enemy.weakPoint.DoubleShockwave());
            amount = 2;
            enemy.GetComponentInChildren<MantisAnimator>().anim.SetTrigger("Double Shockwave");
            firstWaveDelay = 2.0f;
            secondWaveDelay = 0.85f;
        }

        WaveAnim = true;
        yield return new WaitForSeconds(0.7f);
        StartCoroutine(weekpoint.MoveCollider());
        yield return new WaitForSeconds(firstWaveDelay - 0.7f);
        enemy.stop();

        if (!enemy.volnereble) 
        {
            for (int j = 0; j < amount; j++)
            {
                if (!enemy.volnereble)
                {
                    WaveAnim = false;
                    attack.shockwave(shockwavespeed, shockwavezise, shockwaverange);
                    if (j == 0)
                    {
                       //spawning urchins
                       UrchinSpawnerScript.WhichPhaseForUrchin();
                    }
                    yield return new WaitForSeconds(secondWaveDelay);
                    WaveAnim = true;
                }
            }
        }

        WaveAnim = false;
        stunable = false;
        enemy.start();
        StartCoroutine(cooldown(shockwavespeed));
    }

    public IEnumerator elestickdelay(float time)
    {
        StartCoroutine(enemy.weakPoint.Fist());

        ElastickAnim = true;
        SoundcueHandler.PlayFistCue();
        enemy.atta = false;
        stunable = true;
        enemy.GetComponentInChildren<MantisAnimator>().anim.SetTrigger("Punch 0");
        yield return new WaitForSeconds(time);
        ElastickAnim = false;
        if (!enemy.volnereble && !PlayerSlice.SliceMode())
        {
            attack.Elastick(elastickrange, elastickspeed, elastickreturnspeed);
        }
        else
        {
            enemy.atta = true;
            stunable = false;
        }
    }
```
  
</details>

<details>

<summary>Phase 3</summary>

```csharp
 public override void phase()
    {
        if (enemy.Range(startpunchrange) && !enemy.volnereble && !PlayerSlice.SliceMode())
        {
            resetpositon();
        }
        if (enemy.Range(startelastickrange) && enemy.atta && !enemy.volnereble && !PlayerSlice.SliceMode())
        {
            enemy.atta = false;
            if (i % 4 == 0)
            {
                StartCoroutine(elestickdelay(elastickdelai));
            }
            else
            {
                attack.still = true;
                StopCoroutine(dublewave(wavemount));
                wavemount = Random.Range(1, 5);
                StartCoroutine(dublewave(wavemount));
            }
            i++;
            resetpositon();
        }
        else if (!enemy.Range(startpunchrange) && !enemy.volnereble && !PlayerSlice.SliceMode())
        {
            enemy.agent.SetDestination(enemy.target.position);
        }
        else
        {
            resetpositon();
        }
        
    }
```
  
</details>

One problem with it is that everything is not centralized to those scripts. Several important parts of the attacks are in a separate script.The reason is becos we originally did not plan on having multiple phases but by the time we decided to add phases the script was already heavily integrated into multiple systems making it difficult to fully remove. And sadly we did not have time to do that. So it instead became a repository of functions that multiple scripts in the boss uses and a script other systems use to identify the boss.

I originally felt really bad about this project and was thinking that it was all trash. But now after re reading my code I can see that I had a system that would have worked with multiple bosses. Even if it would have needed a major rework. And considering how much the plan changed from week to week I'm actually rather happy with how it came out.



## my other contributions

The tweening of the windows in the main menu










