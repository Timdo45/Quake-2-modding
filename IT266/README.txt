DROP LOOT FEATURE
	CHANGES IN MONSTER FILES
		Added the feature to all monster's die and death functions so that they drop loot
	//IT266 initialized item objects and had a random integer between 0-3 to determine the item. Then called the spawn function.
	edict_t *item
	int random = rand() % 4;
	switch (random){
	case 0:
		item->classname = "item_invulnerability";
		break;
	case 1:
		item->classname = "item_adrenaline";
		break;
	case 2:
		item->classname = "item_armor_jacket";
		break;
	case 3:
		item->classname = "item_pack";
		break;
	}
	ED_CallSpawn(item);
SPAWN WITH EXTRA AMMO AND WEAPONS
	CHANGES IN P_CLIENT.C
		void FetchClientEntData (edict_t *ent)
{
			//IT266 create item object and set it to "shells"
			gitem_t *ammo;
			ammo = FindItem("Shells");

			ent->health = 100;
			ent->max_health = ent->client->pers.max_health;
			ent->flags |= ent->client->pers.savedFlags;
			if (coop->value)
				ent->client->resp.score = ent->client->pers.score;

			//IT266 give player ammo
			Add_Ammo(ent, ammo, 1000);
			ammo = FindItem("Grenades");
			Add_Ammo(ent, ammo, 1000);
			ammo = FindItem("Bullets");
			Add_Ammo(ent, ammo, 1000);
		void InitClientPersistant (gclient_t *client)
{
	//IT266 added item objects
	gitem_t		*item;
	gitem_t		*item2;
	gitem_t		*item3;
	gitem_t		*item4;
	gitem_t		*item5;

	int index;

	memset (&client->pers, 0, sizeof(client->pers));
	
	
	//IT266 Set items to different weapons
	item = FindItem("blaster");
	client->pers.selected_item = ITEM_INDEX(item);
	item2 = FindItem("Shotgun");
	item3 = FindItem("Super Shotgun");
	item4 = FindItem("Machinegun");
	item5 = FindItem("Chaingun");
	//IT266 Give player items
	client->pers.inventory[client->pers.selected_item] = 1;
	client->pers.inventory[ITEM_INDEX(item2)]=2;
	client->pers.inventory[ITEM_INDEX(item3)] = 3;
	client->pers.inventory[ITEM_INDEX(item4)] = 4;
	client->pers.inventory[ITEM_INDEX(item5)] = 5;

RANDOMIZED WEAPON STATS AND WEAPON FEATURES
	CHANGES IN P_WEAPON.C
		void Weapon_Grenade (edict_t *ent)
		|
		|
		|
		// they waited too long, detonate it in their hand
			if (!ent->client->grenade_blew_up && level.time >= ent->client->grenade_time)
			{
				ent->client->weapon_sound = 0;
				weapon_grenade_fire (ent, false);
				ent->client->grenade_blew_up = true;
				//IT266 made player gain health when the grenade explodes in their hand
				ent->health += 120;
			}
		void Blaster_Fire (edict_t *ent, vec3_t g_offset, int damage, qboolean hyper, int effect)
		|
		|
		|
			//IT266 Initialized a random int variable between 0-1999 and set it to 500 if below 500
			int random = rand() % 2000;
			if (random <= 500)
			random = 500;
		|
		|
			//IT266 made the x and y offset vectors random between 0-99 and 0-79 respectivly 	
			VectorSet(offset, rand()%100, rand()%80, ent->viewheight-8);
		|
		|
			fire_blaster (ent, start, forward, damage, random, effect, hyper);
		|
		|
		void Weapon_Blaster_Fire (edict_t *ent)
		//IT266 added new 3D vector object
		vec3_t tempvec;
		|
		|
		//IT266 added random damage and additional blaster fire functions that are fired with the new vector object
		damage = rand()%200;
		if (damage == 0)
		damage = 1;
		Blaster_Fire(ent, vec3_origin, damage, false, EF_BLASTER);
		VectorSet(tempvec, 5, 5, 5);
		VectorAdd(tempvec, vec3_origin, tempvec);
		Blaster_Fire(ent, tempvec, damage, false, EF_BLASTER);

		VectorSet(tempvec, -5, -5, -5);
		VectorAdd(tempvec, vec3_origin, tempvec);
		Blaster_Fire(ent, tempvec, damage, false, EF_BLASTER);
		|
		|
		void Machinegun_Fire (edict_t *ent)
		//IT266 same in this function. Randomized damage as well as randomized kick, vertical spread, and horizontal spread.
		//Also added a command that raised player's health when the gun is fired.
			int			damage = rand()%12;
		if (damage == 0)
			damage = 1;
		int			kick = rand()%200;
		if (kick < 100){
		kick = 100;
		}
		vec3_t		offset;
		int			vspread=rand()%301;
		if (vspread < 100)
		vspread = 100;
		int			hspread = rand() % 501;
		if (hspread < 100)
			hspread = 100;;
		ent->health += 10;
		|
		|
		void weapon_shotgun_fire (edict_t *ent)

		//IT266 added random damage, random vertical, horizontal spread, kick, and number of bullets fired.
			int			damage = rand()%10;
	int			kick =rand()% 200;
	if (kick < 100)
		kick = 100;
	int			vspread = rand() % 501;
	if (vspread < 100)
		vspread = 100;
	int			hspread = rand() % 501;
	if (hspread < 100)
		hspread = 100;;
	int randCount = rand() % 21;
	if (randCount < 5)
		randCount = 5;
	|
	|
	fire_shotgun (ent, start, forward, damage, kick, hspread, vspread, randCount, MOD_SHOTGUN);

		
DIFFERENT MONSTER TYPES FEATURE
	CHANGES IN MONSTER FILES
	void SP_monster_soldier (edict_t *self)
		self->health = rand() % 200;
	if (self->health <= 20)
		self->health = 20;
	self->gib_health = -30;
		
	
DIFFERENT GRENADE TYPES FEATURE
	CHANGES IN G_WEAPON.C
		//IT266 added a new function called Cluster_Grenade_Explode that uses the base grenade explosion function to 
		//fire 4 other grenades that explode. This works because the new grenades are grenades from the grenade launcher
		//This makes it so we don't spawn infinite grenades and crash the game.
		static void Cluster_Grenade_Explode(edict_t *ent)

{
	vec3_t		origin;
	int			mod;

	//IT266 declaring the new grenade vector objects
	vec3_t   grenade1;
	vec3_t   grenade2;
	vec3_t   grenade3;
	vec3_t   grenade4;

	if (ent->owner->client)
		PlayerNoise(ent->owner, ent->s.origin, PNOISE_IMPACT);

	//FIXME: if we are onground then raise our Z just a bit since we are a point?
	if (ent->enemy)
	{
		float	points;
		vec3_t	v;
		vec3_t	dir;

		VectorAdd(ent->enemy->mins, ent->enemy->maxs, v);
		VectorMA(ent->enemy->s.origin, 0.5, v, v);
		VectorSubtract(ent->s.origin, v, v);
		points = ent->dmg - 0.5 * VectorLength(v);
		VectorSubtract(ent->enemy->s.origin, ent->s.origin, dir);
		if (ent->spawnflags & 1)
			mod = MOD_HANDGRENADE;
		else
			mod = MOD_GRENADE;
		T_Damage(ent->enemy, ent, ent->owner, dir, ent->s.origin, vec3_origin, (int)points, (int)points, DAMAGE_RADIUS, mod);
	}

	if (ent->spawnflags & 2)
		mod = MOD_HELD_GRENADE;
	else if (ent->spawnflags & 1)
		mod = MOD_HG_SPLASH;
	else
		mod = MOD_G_SPLASH;
	T_RadiusDamage(ent, ent->owner, ent->dmg, ent->enemy, ent->dmg_radius, mod);

	VectorMA(ent->s.origin, -0.02, ent->velocity, origin);
	gi.WriteByte(svc_temp_entity);
	if (ent->waterlevel)
	{
		if (ent->groundentity)
			gi.WriteByte(TE_GRENADE_EXPLOSION_WATER);
		else
			gi.WriteByte(TE_ROCKET_EXPLOSION_WATER);
	}
	else
	{
		if (ent->groundentity)
			gi.WriteByte(TE_GRENADE_EXPLOSION);
		else
			gi.WriteByte(TE_ROCKET_EXPLOSION);
	}
	gi.WritePosition(origin);
	gi.multicast(ent->s.origin, MULTICAST_PHS);



	G_FreeEdict(ent);

	//IT266 setting our vector objects
	VectorSet(grenade1, 10, 20, 50);
	VectorSet(grenade2, 10, -20, 50);
	VectorSet(grenade3, -10, 20, 50);
	VectorSet(grenade4, -10, -20, 50);
	//IT266 firing the grenades.
	fire_grenade(ent, origin, grenade1, 120, 10, 1.0, 120, true);
	fire_grenade(ent, origin, grenade2, 120, 10, 1.0, 120, true);
	fire_grenade(ent, origin, grenade3, 120, 10, 1.0, 120, true);
	fire_grenade(ent, origin, grenade4, 120, 10, 1.0, 120, true);

	G_FreeEdict(ent);
} 
	void fire_grenade2 (edict_t *self, vec3_t start, vec3_t aimdir, int damage, int speed, float timer, float damage_radius, qboolean held)
	|
	|
		//IT266 has the hand grenade call the Cluster_Grenade_Explode function.
		grenade->think = Cluster_Grenade_Explode;