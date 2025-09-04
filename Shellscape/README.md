# Shellscape
Enemy Programer

Duration: 11/2024 - 01/2025

Engine: Unity

Genre: First Person Shoter

Team: 3 Programers, 3 Artists

# My contribution
I was primaraly in charge of the enemy

the entire project was plagued by problems. which resulted in a lot of janky code and janky game play. if I culd go back I whuld defenstrate all the skripts about the enemy and restart.
most problems came from bad planing and a ever changing scope. the game works and the enemy funktions just very janky. I did lern a lot from this project which defenetly helped in later projects.

enouth whit my doom and gloom time to describe the hideus spagety that shuld be defenstrated.

in the original consept for the game there was supose to be 3 bosses. one for each programer and since I was supose to make the first one I tried to create a modular system that the others culd use for there bosses.
Do to that I tried to use inheritence to make it easy but it ended upp over complicating the scripts since some are 4 inherentes deep. the single some what good use was for the phase system.

I used a state machene to determine the phase the boss was on. For some reason I made it in the uppdate funktion instead of making a seperet funktion that only run when the boss takes damage.

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

The phases themselves inherited from a script that held all the boss's attacks wich inherits from a BasePhase script.




