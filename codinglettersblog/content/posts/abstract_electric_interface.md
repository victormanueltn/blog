---
title: "Abstract electric interface"
date: 2024-02-11T12:54:16+01:00
draft: true
---

Dear reader,

I was recently listening to the Our Electrified Future episode of the StarTalk Radio podcast and they mentioned two facts: 1. Electric cars consume, well, electricity. 2. Electricity can be generated in many different ways. Examined individually, each fact is somewhat elementary, together they are more interesting. And yes, it has something to do with software.

Gasoline cars have a single fuel choice and gasoline comes from oil. That's it. If you want to change your fuel, you have to change your engine, which sounds a lot like changing your car. Electricity, on the other hand, can be generated from various sources such as solar, wind, nuclear, hydroelectric power, or even by burning fuel. Electric cars have no knowledge of the origin of the electricity they use. This is the key idea.

When I listened to the StarTalk Radio episode, I found these ideas interesting and oddly familiar... and for good reason! As a developer, I am familiar with using an abstraction and not worrying about the details behind it. Let's unpack this with a bit of code:

``` rs
pub trait TransformToElectricity {
    fn transform(&self) -> String;
}

// Define a struct that represents a resource
pub struct SolarEnergy;

// Implement the trait for the resource
impl TransformToElectricity for SolarEnergy {
    fn transform(&self) -> String {
        String::from("Transforming solar energy to electricity")
    }
}

// Define the Car struct
pub struct Car<T: TransformToElectricity> {
    resource: T,
}

// Implement a method to use the resource
impl<T: TransformToElectricity> Car<T> {
    pub fn use_resource(&self) -> String {
        self.resource.transform()
    }
}

// Usage
let solar_panel = SolarPanel;
let car = Car { resource: solar_panel };
println!("{}", car.use_resource());  // Outputs: "Transforming solar energy to electricity"

```
